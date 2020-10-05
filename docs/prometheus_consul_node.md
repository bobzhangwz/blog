# Node Exporter 通过 consul 上报到 prometheus

## 原理

consul 本身提供了提供了服务的健康检查功能（[Service Health Check](https://www.consul.io/docs/agent/checks.html)); 我们可以利用服务健康检查来上报 Node Exporter 信息。

## 安装 node_exporter

* 下载 https://github.com/prometheus/node_exporter/releases/download/0.12.0/node_exporter-0.12.0.linux-amd64.tar.gz
* 将文件解压到 `/usr/local/bin/node_exporter`
* 编写 systemd 脚本 管理 node_exporter 启停， `/usr/lib/systemd/system/node-exporter.service`

```
[Unit]
Description=prometheus node exporter bin

[Service]
Restart=on-failure
RestartSec=20
TimeoutStartSec=20m
ExecStart=/usr/local/bin/node_exporter -web.listen-address=:9100   # 监听9100端口

[Install]
WantedBy=multi-user.target
```
* 运行命令 `systemctl daemon-reload` 和 `service node-exporter start`, 就可以访问 `http://agent:9100/metrics` 来查看 Exporter 导出信息

## 设置 consul service 检查 node-exporter 服务

> 请先按照 上一篇文章 安装好 consul

* 添加文件 `/etc/consul/node_exporter.json`， 定义 node_exporter 服务，并使 consul 对 node-exporter 进行健康检查

```
{
    "service": {
        "id": "node_exporter",
        "name": "node_exporter",
        "check": {
            "http": "http://127.0.0.1:9100",
            "interval": "5s",
            "timeout": "2s"
        }
    }
}
```

* 打开 `http://consul-master:8500` 可以看到 服务已经起来了
![image_1anhq36lhnov1lej1sq24st17j29.png-30.1kB][1]
* 如果将 node-exporter 关闭 `service node-exporter stop`，可以看到 node-exporter 会这样显示
![image_1anhq5vds5rm16o0183j1l3sobcm.png-28.4kB][2]


  [1]: http://static.zybuluo.com/zhpooer/xblhpum82l49lf08il5i6c8f/image_1anhq36lhnov1lej1sq24st17j29.png
  [2]: http://static.zybuluo.com/zhpooer/k3jqw0hxhsx3kiyvi0vqnoxr/image_1anhq5vds5rm16o0183j1l3sobcm.png
