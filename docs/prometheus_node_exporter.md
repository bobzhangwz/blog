# Prometheus Exporter 之 Node Exporter

[prometheus/node_exporter](https://github.com/prometheus/node_exporter) 是 prometheus 官方提供的一个收集机器性能的一个 Exporter

## 运行 Node Exporter

运行 Node Exporter 有两种方式

1. 使用 docker 来运行 Node Exporter，但是因为 Node Exporter 是运行在容器里面，以至于获取到的系统运行参数可能会有偏差。
2. 直接使用二进制 Node Exporter 在宿主机子上运行

### 在docker中运行 Node Exporter

```
/usr/bin/docker run \
    --name=node-exportor \
    -p {{node_exportor_port}}:9100 \
    -v "/proc:/host/proc" \
    -v "/sys:/host/sys" \
    -v "/:/rootfs" \
    --net="host" \
    prom/node-exporter:latest \
    -collector.procfs /host/proc \
    -collector.sysfs /host/proc \
    -collector.filesystem.ignored-mount-points "^/(sys|proc|dev|host|etc)($|/)"
```
参考文章： https://www.digitalocean.com/community/tutorials/how-to-install-prometheus-using-docker-on-ubuntu-14-04

相关 Ansible 脚本： https://github.com/peterwangpei/mesos-poc/tree/master/prod/prometheus/roles/node-exportor

### 直接在机器是运行 Node Exporter

```
./node_exporter <flags>
```

 Ansible 自动化安装脚本：https://github.com/peterwangpei/mesos-poc/blob/master/prod/prometheus/roles/node-exporter-bin/tasks/main.yml

## 获取需要参数



| 指标         | 业务含义                                         | exporter 标签                      |
|----------------------------------------------------------------------------------------------------|
| cpu.idle     | 当前 cpu 空闲时间                                | node_cpu{cpu="cpu0",mode="idle"}   |
| cpu.iowait   | 当前 cpu 处在等待 IO 操作的时间                  | node_cpu{cpu="cpu0",mode="iowait"} |
| cpu.stolen   | 虚拟机等待分配时间                               | node_cpu{cpu="cpu0",mode="steal"}  |
| cpu.system   | 当前 cpu 运行内核所占时间                        | node_cpu{cpu="cpu0",mode="system"} |
| cpu.user     | 当前 cpu 运行用户进程所占时间                    | node_cpu{cpu="cpu0",mode="user"}   |
| disk.free    | 空余磁盘空间                                     | node_filesystem_free               |
| disk.total   | 磁盘总大小                                       | node_filesystem_size               |
| inodes.free  | Filesystem total free file nodes.                | node_filesystem_files_free         |
| indoes.total | Filesystem total file nodes.                     | node_filesystem_files              |
|              | The number of I/Os currently in progress         | node_disk_io_now                   |
|              | The weighted # of milliseconds spent doing I/Os. | node_disk_io_time_weighted         |
|              | Milliseconds spent doing I/Os.                   | node_disk_io_time_ms               |
| load.1       | 1分钟内系统负载                                  | node_load1                         |
| load.5       | 5分钟内系统负载                                  | node_load5                         |
| load.15      | 15分钟内系统负载                                 | node_load15                        |
| mem.free     | 可用内存量                                       | node_memory_MemFree                |
| mem.total    | 总内存                                           | node_memory_MemTotal               |
