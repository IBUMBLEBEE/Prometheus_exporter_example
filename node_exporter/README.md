# node_exporter

Node Exporter 是\*NIX 内核系统和硬件系统指标的 Prometheus exporter，用 Go 编写，带可插拔的指标收集器。

对于 Windows 用户，推荐使用 [Windows Exporter](https://github.com/prometheus-community/windows_exporter)。

## 安装

下载 [node_exporter](https://prometheus.io/download/#node_exporter)。

```shell
# 创建可配置的文件
# OPTIONS="--web.disable-exporter-metrics --collector.textfile.directory /var/lib/node_exporter/textfile_collector --collector.filesystem.ignored-fs-types ^(autofs|binfmt_misc|bpf|cgroup2?|configfs|debugfs|devpts|devtmpfs|fusectl|hugetlbfs|mqueue|nsfs|overlay|proc|procfs|pstore|rpc_pipefs|securityfs|selinuxfs|squashfs|sysfs|tracefs|tmpfs|dm-)$ --collector.netdev.ignored-devices ^(veth|docker).* --collector.netclass.ignored-devices ^(veth|docker).* --collector.diskstats.ignored-devices ^(ram|loop|fd|dm-|(h|s|v|xv)d[a-z]+|nvme[0-9]+n[0-9]+p)[0-9]+$ --collector.systemd --collector.systemd.unit-whitelist ^(docker|mysql|rabbitmq|redis|cassandra).*"
touch /etc/sysconfig/node_exporter
cat >/etc/systemd/system/node_exporter.service<<EOF
[Unit]
Description=Node Exporter

[Service]
User=prometheus
EnvironmentFile=/etc/sysconfig/node_exporter
ExecStart=/usr/sbin/node_exporter --web.listen-address=localhost:9100
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

## Textfile Collector

Textfile Collector 采集自定义的指标。textfile 类似于[Pushgateway](https://github.com/prometheus/pushgateway)的收集器，这个收集器可以允许我们暴露自定义指标。

配置的脚本地址：[https://github.com/prometheus-community/node-exporter-textfile-collector-scripts](https://github.com/prometheus-community/node-exporter-textfile-collector-scripts)

## 详细信息

[https://github.com/prometheus/node_exporter](https://github.com/prometheus/node_exporter)
