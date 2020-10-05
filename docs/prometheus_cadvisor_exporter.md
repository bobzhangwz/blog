# Prometheus Exporter 之 cAdvisor

[cAdvisor (Container Advisor)](https://github.com/google/cadvisor) 可以为容器使用者提供容器的资源使用情况和性能数据，是一个集收集、聚合、处理以及导出容器运行信息功能的一个后台进程。

cAdvisor 可以运行于容器里面，运行命令如下

```
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest

```

因为 kubernetes 的 kubelet 已经集成了 cadvisor，所以可以直接访问 `4194` 端口访问 cAdvisor。同时 cAdvisor 原生提供了 Prometheus Exporter 的接口，所以 Prometheus 可以自己从 cAdvisor 上获取 docker容器 的运行信息，访问链接 `http://cAdvisor_host:4194/metrics`。

## cAdvisor 提供的性能数据

|  prometheus 标签  |       业务含义        |
|--|--|
|                               cadvisor_version_info |  A metric with a constant '1' value labeled by kernel version, OS version, docker version, cadvisor version & cadvisor revision. |
|                  container_cpu_system_seconds_total |  Cumulative system cpu time consumed in seconds. |
|                   container_cpu_usage_seconds_total |  Cumulative cpu time consumed per cpu in seconds. |
|                    container_cpu_user_seconds_total |  Cumulative user cpu time consumed in seconds. |
|                             container_fs_io_current |  Number of I/Os currently in progress |
|                  container_fs_io_time_seconds_total |  Cumulative count of seconds spent doing I/Os |
|         container_fs_io_time_weighted_seconds_total |  Cumulative weighted I/O time in seconds |
|                            container_fs_limit_bytes |  Number of bytes that can be consumed by the container on this filesystem. |
|                     container_fs_read_seconds_total |  Cumulative count of seconds spent reading |
|                     container_fs_reads_merged_total |  Cumulative count of reads merged |
|                            container_fs_reads_total |  Cumulative count of reads completed |
|                     container_fs_sector_reads_total |  Cumulative count of sector reads completed |
|                    container_fs_sector_writes_total |  Cumulative count of sector writes completed |
|                            container_fs_usage_bytes |  Number of bytes that are consumed by the container on this filesystem. |
|                    container_fs_write_seconds_total |  Cumulative count of seconds spent writing |
|                    container_fs_writes_merged_total |  Cumulative count of writes merged |
|                           container_fs_writes_total |  Cumulative count of writes completed |
|                                 container_last_seen |  Last time a container was seen by the exporter |
|                            container_memory_failcnt |  Number of memory usage hits limits |
|                     container_memory_failures_total |  Cumulative count of memory allocation failures. |
|                        container_memory_usage_bytes |  Current memory usage in bytes. |
|                  container_memory_working_set_bytes |  Current working set in bytes. |
|               container_network_receive_bytes_total |  Cumulative count of bytes received |
|              container_network_receive_errors_total |  Cumulative count of errors encountered while receiving |
|     container_network_receive_packets_dropped_total |  Cumulative count of packets dropped while receiving |
|             container_network_receive_packets_total |  Cumulative count of packets received |
|              container_network_transmit_bytes_total |  Cumulative count of bytes transmitted |
|             container_network_transmit_errors_total |  Cumulative count of errors encountered while transmitting |
|    container_network_transmit_packets_dropped_total |  Cumulative count of packets dropped while transmitting |
|            container_network_transmit_packets_total |  Cumulative count of packets transmitted |
|                              container_scrape_error |  1 if there was an error while getting container metrics, 0 otherwise |
|                           container_spec_cpu_shares |  CPU share of the container. |
|                   container_spec_memory_limit_bytes |  Memory limit for the container. |
|              container_spec_memory_swap_limit_bytes |  Memory swap limit for the container. |
|                        container_start_time_seconds |  Start time of the container since unix epoch in seconds. |
|                               container_tasks_state |  Number of tasks in given state |
|                                     get_token_count |  Counter of total Token() requests to the alternate token source |
|                                get_token_fail_count |  Counter of failed Token() requests to the alternate token source |
|                              go_gc_duration_seconds |  A summary of the GC invocation durations. |
|                                       go_goroutines |  Number of goroutines that currently exist. |
|                  http_request_duration_microseconds |  The HTTP request latencies in microseconds. |
|                             http_request_size_bytes |  The HTTP request sizes in bytes. |
|                                 http_requests_total |  Total number of HTTP requests made. |
|                            http_response_size_bytes |  The HTTP response sizes in bytes. |
|      kubelet_container_manager_latency_microseconds |  Latency in microseconds for container manager operations. Broken down by method. |
|                    kubelet_containers_per_pod_count |  The number of containers per pod. |
|                               kubelet_docker_errors |  Cumulative number of Docker errors by operation type. |
|      kubelet_docker_operations_latency_microseconds |  Latency in microseconds of Docker operations. Broken down by operation type. |
|    kubelet_generate_pod_status_latency_microseconds |  Latency in microseconds to generate status for a single pod. |
|              kubelet_pod_start_latency_microseconds |  Latency in microseconds for a single pod to go from pending to running. Broken down by podname. |
|             kubelet_pod_worker_latency_microseconds |  Latency in microseconds to sync a single pod. Broken down by operation type: create, update, or sync |
|       kubelet_pod_worker_start_latency_microseconds |  Latency in microseconds from seeing a pod to starting a worker. |
|                     kubelet_running_container_count |  Number of containers currently running |
|                           kubelet_running_pod_count |  Number of pods currently running |
|              kubelet_sync_pods_latency_microseconds |  Latency in microseconds to sync all pods. |
|                                   machine_cpu_cores |  Number of CPU cores on the machine. |
|                                machine_memory_bytes |  Amount of memory installed on the machine. |
|                           process_cpu_seconds_total |  Total user and system CPU time spent in seconds. |
|                                     process_max_fds |  Maximum number of open file descriptors. |
|                                    process_open_fds |  Number of open file descriptors. |
|                       process_resident_memory_bytes |  Resident memory size in bytes. |
|                          process_start_time_seconds |  Start time of the process since unix epoch in seconds. |
|                        process_virtual_memory_bytes |  Virtual memory size in bytes. |
|                               ssh_tunnel_open_count |  Counter of ssh tunnel total open attempts |
|                          ssh_tunnel_open_fail_count |  Counter of ssh tunnel failed open attempts |
