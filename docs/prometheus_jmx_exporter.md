# Prometheus Exporter 之 JMX Exporter

[JMX Exporter](https://github.com/prometheus/jmx_exporter) 是 Prometheus 官方提供的一个 Exporter

JMX Exporter 的工作原理是通过 Java虚拟机的 JMX 框架，来读取 Java虚拟机的状态，并导出成 Prometheus 的格式，同时也可以根据一些规则，进行定制化输出监控信息。

所以要使用 JMX Exporter 监控 Tomcat 的运行状态，必须先开启 Tomcat 的 JMX 端口。

开启 Tomcat JMX 端口需要配置Tomcat的启动参数

    CATALINA_OPTS="-Dcom.sun.management.jmxremote  -Dcom.sun.management.jmxremote.port={{jmx_port}} -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.local.only=false"

> jmx_port 指定 Jmx 通过哪个端口提供服务

## 启动JMX Exporter

根据官方文档，JMX Exporter 是通过 jar 包来启动的。在使用容器的环境里，JMX Exporter 要获取 Tomcat 运行参数，可以有两种方式

 1. 将 JMX Exporter 单独做一个容器，JMX Exporter 容器读取 Tomcat 暴露出的 JMX 端口，来获取 Tomcat 运行参数。
 2. 让 JMX Exporter 和 Tomcat 在一个容器里面运行，并使用 supervisor 来启动两个进程。此方案不符合 docker 的设计原则，不推荐使用。


### 方式一 JMX Exporter 当做单独容器

因为 prometheus 官方没有提供 JMX Exporter 的 docker 容器，但是有第三方的容器可供使用 https://github.com/yagniio/docker-jmx-exporter

- 先启动 tomcat 容器

        docker run -d \
        -p 8888:8080 \
        -p {{jmx_port}}:{{jmx_port}} \
        --name tomcat \
        --env CATALINA_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port={{jmx_port}} -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.local.only=false" \
        tomcat:8.0

- 启动 jmx-exporter

        docker run -d \
        -v /etc/prometheus/exporter:/jmx_prometheus \
        -p {{exporter_port}}:9138 \
        --name jmx_exporter \
        yagni/docker-jmx-exporter:0.7-SNAPSHOT /jmx_prometheus/tomcat.yml

- 配置文件 `tomcat.yml`

        # 连接到 Tomcat 的 JMX 端口，并输出全部的参数
        hostPort: "{{docker_host}}:{{jmx_port}}"
        lowercaseOutputLabelNames: true
        lowercaseOutputName: true

### 方式二 JMX Exporter 与 tomcat 运行于同一个容器

因为没有第三方的容器所以需要自己写 Dockerfile 来构建

- `Dockerfile`

```
FROM tomcat:7.0-jre7

ENV JMX_PROMETHEUS_HTTPSERVER_VERSION 0.7-SNAPSHOT

ENV CATALINA_OPTS "-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8880 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.local.only=false"

RUN apt-get update && apt-get install -y curl supervisor

RUN mkdir -p /var/log/supervisor

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

RUN mkdir -p /jmx_prometheus_httpserver /jmx_prometheus

COPY tomcat.yml /jmx_prometheus/tomcat.yml

WORKDIR /jmx_prometheus_httpserver

RUN curl -O -k -L https://github.com/yagniio/docker-jmx-exporter/releases/download/$JMX_PROMETHEUS_HTTPSERVER_VERSION/jmx_prometheus_httpserver-$JMX_PROMETHEUS_HTTPSERVER_VERSION-jar-with-dependencies.jar

EXPOSE 8080 9138

CMD ["/usr/bin/supervisord"]

```
- `supervisord.conf`:

```
[supervisord]
nodaemon=true
pidfile=/var/run/supervisord.pid;
loglevel=debug

[program:tomcat]
command=/usr/local/tomcat/bin/catalina.sh run
redirect_stderr=true

[program:jmx_exporter]
command=bash -c "java ${JVM_OPTS:--Xmx256m} -jar /jmx_prometheus_httpserver/jmx_prometheus_httpserver-*.jar 9138 /jmx_prometheus/tomcat.yml"
redirect_stderr=true
```
- `tomcat.yml`

```
---
hostPort: "localhost:8880"
lowercaseOutputLabelNames: true
lowercaseOutputName: true
```

运行命令 `docker build -t prometheus/tomcat_exporter` 构建，并运行容器 `docker run -p 9138:9138 -p 8080:8080 -d prometheus/tomcat_exporter`

## 定制 tomcat exporter 信息

我们只关心 jvm 的一些参数，定制的文件 `tomcat.yml` 如下

```
---
hostPort: "localhost:8880"
lowercaseOutputLabelNames: true
lowercaseOutputName: true

rules:
  # 获取内存信息
  - pattern: 'java.lang<type=Memory><(\w+)>(\w+)'
    name: tomcat_memory_$1_$2
    labels:
      type: "$1"
    type: GAUGE

  # 获取线程信息
  - pattern: 'Catalina<type=ThreadPool, name="(\w+-\w+)-(\d+)"><>(currentThreadCount|currentThreadsBusy|keepAliveCount|pollerThreadCount|connectionCount):'
    name: tomcat_threadpool_$3
    labels:
      port: "$2"
      protocol: "$1"
    type: COUNTER

  # 获取请求信息
  - pattern: 'Catalina<type=GlobalRequestProcessor, name=\"(\w+-\w+)-(\d+)\"><>(\w+):'
    name: tomcat_$3_total
    labels:
      port: "$2"
      protocol: "$1"
    help: Tomcat global $3
    type: COUNTER

  # 获取垃圾收集信息
  - pattern: 'java.lang<type=GarbageCollector, name=(\w+)><(\w*)>(\w+)'
    name: tomcat_gc_$2_$3
    labels:
      name: "$1"
    type: GAUGE

```

通过 prometeus 的查询语句，可以查询一下内容，请自定义相关标签，请配合配置文件`tomcat.yml`使用：

> 1. 当前线程数目，`tomcat_threadpool_currentthreadcount`
2. 已发生的垃圾收集总数，
    1. 标记清除算法收集次数`tomcat_gc_collectioncount{name="MarkSweepCompact"}`
    2. 复制引用算法次数 `tomcat_gc_collectioncount{name="Copy"}`
3. 累计垃圾收集时间
    1. 标记清除算法收集所用时间`tomcat_gc_collectiontime{name="MarkSweepCompact"`
    2. 复制引用算法所用时间 `tomcat_gc_collectiontime{name="Copy"}`
4. 总的堆内存 `tomcat_memory_heapmemoryusage_used`
5. 所用的总 java 堆内存提交 `tomcat_memory_heapmemoryusage_committed`
6. 初始堆内存 `tomcat_memory_heapmemoryusage_init`
7. 最大堆内存 `tomcat_memory_heapmemoryusage_max`
8. 总的非堆内存 `tomcat_memory_heapmemoryusage_committed`
9. 所用的总 java 非堆内存提交 `tomcat_memory_heapmemoryusage_committed`
10. 初始非堆内存 `tomcat_memory_heapmemoryusage_committed`
11. 最大非堆内存 `tomcat_memory_heapmemoryusage_committed`

还可以定制其他的性能参数收集，比如 CPU参数、session个数、用户请求数，请根据配置文件配置。
