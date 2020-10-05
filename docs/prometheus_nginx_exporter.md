# Prometheus Exporter 之 Nginx Exporter

Prometheus 官方没有提供 nginx 的 Exporter，但是有第三方组织提供了 nginx 的 Exporter，有两种方式：

1. 在nginx中使用 lua 的 prometheus类库，来获取nginx的性能参数。此方法的好处是 通过 lua 来嵌入 nginx 执行来获取性能参数，只需要执行一个 nginx 进程就可以提供 prometheus 参数。但是需要先安装 ngnix-extra 来支持 lua 运行环境，同时要在自己编写 lua 代码来采集参数，这需要熟悉的 lua 开发人员来配置。[nginx metric lib](https://github.com/knyar/nginx-lua-prometheus)  [Dockerfile](https://github.com/peterwangpei/mesos-poc/tree/master/prod/prometheus/nginx_exporter_builder)
2. nginx 启用 `ngx_http_stub_status_module` 模块，同时运行 (nginx-exporter)[https://hub.docker.com/r/fish/nginx-exporter/] 容器，来读取 nginx 的运行参数，并转化为 prometheus 可被读取的格式。这种方式，安装方便，比较建议使用此方式获取性能参数。[此地址](https://github.com/peterwangpei/mesos-poc/tree/master/prod/prometheus/roles/nginx-exporter) 可查看 ansible 运行一套 nginx_exporter 的例子。

## 安装方式 ##

这里提供了方式二的 Niginx Exporter 的安装方式，方式一请查看相关链接。

因为 我们需要 nginx 和 nginx exporter 都同时跑到一个容器里面，但是 nginx exporter 是作为单独的一个进程跑在 docker 里面的，所以要使用 supervisord 将nginx 和 nginx exporter 运行到 一个 docker 里面

[参考网址](
https://github.com/peterwangpei/mesos-poc/tree/master/prod/prometheus/nginx_exporter_builder/nginx_exporter)


- Dockerfile

```
FROM centos:7

RUN yum update -y && yum install -y epel-release

RUN yum install -y nginx supervisor

RUN mkdir -p /var/log/supervisor

COPY supervisord.conf /etc/supervisord.conf

COPY nginx_status.conf /etc/nginx/conf.d/

COPY nginx_exporter /bin

EXPOSE 80 9113

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
```

- `nginx_status.conf`

```
server {
  listen       9191;
  server_name  localhost;
  location /nginx_status {
    stub_status on;
    access_log off;
  }
}
```

- `supervisord.conf`

```
[supervisord]
nodaemon=true
pidfile=/var/run/supervisord.pid;
loglevel=debug

[program:nginx]
command=bash -c "nginx -g 'daemon off;'"
redirect_stderr=true

[program:nginx_exporter]
command=/bin/nginx_exporter -nginx.scrape_uri=http://localhost:9191/nginx_status
redirect_stderr=true
```

- [nginx_exporter](https://github.com/peterwangpei/mesos-poc/raw/master/prod/prometheus/nginx_exporter_builder/nginx_exporter/nginx_exporter)

> 运行命令 `docker build -t k8s:nginx_exporter .`


## Nginx 提供的性能参数 ##

|  prometheus 标签  |       业务含义        |
|----|----|
|                              go_gc_duration_seconds |  A summary of the GC invocation durations. |
|                                       go_goroutines |  Number of goroutines that currently exist. |
|                  http_request_duration_microseconds |  The HTTP request latencies in microseconds. |
|                             http_request_size_bytes |  The HTTP request sizes in bytes. |
|                                 http_requests_total |  Total number of HTTP requests made. |
|                            http_response_size_bytes |  The HTTP response sizes in bytes. |
|                           nginx_connections_current |  Number of connections currently processed by nginx |
|                   nginx_connections_processed_total |  Number of connections processed by nginx |
|                           process_cpu_seconds_total |  Total user and system CPU time spent in seconds. |
|                                     process_max_fds |  Maximum number of open file descriptors. |
|                                    process_open_fds |  Number of open file descriptors. |
|                       process_resident_memory_bytes |  Resident memory size in bytes. |
|                          process_start_time_seconds |  Start time of the process since unix epoch in seconds. |
|                        process_virtual_memory_bytes |  Virtual memory size in bytes. |
