# 使用 consul agent 启停prometheus exporter

## 原理

consul 本身提供了提供了服务的健康检查功能（[Service Health Check](https://www.consul.io/docs/agent/checks.html)); 同时配置 consul 检查应用(tomcat)的端口，当端口有效时，触发 [consul watcher](https://www.consul.io/docs/agent/watches.html)，调用指定的脚本(启停prometheus exporter)。

## 最小化验证

> 以下配置只针对一个tomcat服务，如果要有多个 tomcat 服务，需要将配置复制一遍，并修改起 端口，服务名等配置

### 一. 在虚拟机上启动 consul agent

* 在 `/etc/consul/consul.json` 配置 consul 启动参数

```
{
  "data_dir": "/var/lib/consul",
  "advertise_addr": "192.168.33.167", ## 本地 ip
  "client_addr": "0.0.0.0",
  "bind_addr": "192.168.33.167",      ## 本地 ip
  "ui": true,
  "retry_join": ["192.168.33.156"],   ## 接入 consul master 的机器
  "log_level": "INFO"
}
```

* 运行consul，并指定 config-dir 为 `/etc/consul`

```
/usr/bin/consul agent -config-dir=/etc/consul
```

也可以使用 systemd 配置启动，添加文件`/etc/systemd/system/consul.service`，并运行命令 `systemctl daemon-reload`，具体内容如下

```
[Unit]
Description=Consul is a tool for service discovery and configuration.
Documentation=https://consul.io
After=network-online.target
Wants=network-online.target
After=rsyslog.service
Wants=rsyslog.service

[Service]
User=consul   ## 可以在这里指定启动consul的用户和用户组
Group=consul
ExecStart=/usr/bin/consul agent -config-dir=/etc/consul
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

### 二. 配置consul对 tomcat 8080端口进行健康检查

* 在 `/etc/consul` 文件夹下创建文件 `tomcat-service.json`（文件名任意）

```
{
  "service": {
    "id": "tomcat",
    "name": "tomcat",                   # 指定服务名，可以设定为 prometheus
    "tags": ["master"],                 # 指定各种服务参数
    "address": "127.0.0.1",
    "port": 6000,                       # 可以设定为 prometheus 的端口
    "check": {
      "http": "http://localhost:8080",  # 检查8080端口，是否可用
      "interval": "5s",                 # 检查间隔
      "timeout": "2s"
    }
  }
}
```

* 重启 consul，访问 http://consul-master:8500，可以看到一个tomcat服务，但是状态为 `service:tomcat critical`

![image_1anheo4k049l8vnmu1ua8517m.png-22.9kB][1]

### 三. 编写 tomcat prometheus exporter 系统服务

每一个 tomcat实例，都需要配一个 prometheus exporter 系统服务，并开放不同的exporter端口, 以便第四步的监听程序调用

* 下载 `jmx_exporter.jar` `curl -O -k -L https://github.com/yagniio/docker-jmx-exporter/releases/download/0.7-SNAPSHOT/jmx_prometheus_httpserver-0.7-SNAPSHOT-jar-with-dependencies.jar` 到 `/etc/prometheus/prom.jar`

* 配置 `/etc/prometheus/tomcat.yml`

```
---
hostPort: "localhost:8321"       ## tomcat 暴露出去的 jmx 端口
lowercaseOutputLabelNames: true
lowercaseOutputName: true
```

* 添加文件 `/usr/lib/systemd/system/tomcat-exporter.service`, 并运行 `systemctl daemon-reload`

```
[Unit]
Description=tomcat

[Service]
Restart=on-failure
RestartSec=20
TimeoutStartSec=20m

ExecStart=/usr/bin/java -Xmx125m -jar /etc/prometheus/prom.jar 9138 /etc/prometheus/tomcat.yml  ### 9138 为 exporter 端口

[Install]
WantedBy=multi-user.target
```

### 四. 配置 consul watcher 监听服务变化，调用prometheus exporter起停脚本

* 添加脚本`/etc/prometheus/watch_tomcat` `chmod 0755 watcher_tomcat`

```
curl -sSf http://localhost:8080 > /dev/null    ## 获取服务状态
if [ $? = 0 ]; then
  service tomcat-exporter start   # 需要执行权限
else
  service tomcat-exporter stop
fi
```

> 注意这个脚本会被 consul 执行，执行这个脚本的用户为 consul进程的用户，所以要要注意脚本里边命令在执行权限，如果没有权限，脚本将不能成功执行；可以通过 `sudo service consul status` 来查看 consul 的错误信息

* 设置 consul 的 watchers，添加文件 `/etc/consul/consul-watchers.json`

```
{
  "watches": [{
    "type": "service",
    "service": "tomcat",
    "handler": "/etc/prometheus/watch_tomcat"
  }]
}
```

> 设置监听服务名称，以及服务变化时，执行的脚本

### 五. 启动tomcat服务，监听8080端口，查看consul状态

* 通过 docker 启动 tomcat 暴露8080端口，并启用 jmx

```
docker run -d \
-p 8080:8080 \
-p {{jmx_port}}:{{jmx_port}} \
--name tomcat \
--env CATALINA_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port={{jmx_port}} -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.local.only=false" \
tomcat:8.0
```

* 可以查看 `http://consul-master:8500`, tomcat服务状态自动变为 `service:tomcat passing`

![image_1anhema0k14du1uuk16stlpc19tn9.png-14.5kB][2]

* 执行`service tomcat-exporter status`可以看到，`tomcat-exporter` 会自动启动；如果将 tomcat 服务关闭，执行`service tomcat-exporter status`可以看到，`tomcat-exporter`服务会自动停止


  [1]: http://static.zybuluo.com/zhpooer/5l7lw34y5fbj603ynv9dnaav/image_1anheo4k049l8vnmu1ua8517m.png
  [2]: http://static.zybuluo.com/zhpooer/83tx6rm10q6itesyin0xtzef/image_1anhema0k14du1uuk16stlpc19tn9.png
