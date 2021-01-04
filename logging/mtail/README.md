# Google's mtail log data extractor - mtail

mtail是用于从应用程序日志中提取指标以将导出到时序数据库或时间序列计算器以进行警报和显示的工具。如从日志中解析出需要的数据，生成Prometheus需要的格式。详细信息请查看：[mtail](https://github.com/google/mtail)。

## 安装

```shell
wget https://github.com/google/mtail/releases/download/v3.0.0-rc38/mtail_v3.0.0-rc38_linux_amd64 -O /usr/local/bin/mtail
chmod +x /usr/local/bin/mtail
# 写入.service启动文件
cat >/etc/systemd/system/mtail.service<<EOF
[Unit]
Description=mtail

[Service]
EnvironmentFile=/etc/default/mtail
ExecStart=/usr/local/bin/mtail $ARGS

Restart=always
OOMScoreAdjust=1000

[Install]
WantedBy=multi-user.target
EOF

#service的省缺参数，注意设置日志目录
cat >/etc/default/mtail<<EOF
ARGS="-progs /etc/mtail -logs /var/log/message"
EOF
```

```txt
说明：
OOMScoreAdjust：设置进程因内存不足而被杀死的优先级。 可设为 -1000(禁止被杀死) 到 1000(最先被杀死)之间的整数值。
```

## 配置

配置解析日志的规则，在官方的example中有各种常用的解析规则。如apache。
详情请查看：[mtail example](https://github.com/google/mtail/tree/master/examples)。

如有自定义日志格式，可以使用在线正则表达式解析，[在线正则](https://regex101.com/)。mtail使用Golang开发，正则表达式符合`Golang RE2`，在线配置的时候请选择golang 匹配模式。详细的[Golang RE2的语法](https://github.com/google/re2/wiki/Syntax)。

在指定配置的目录下生成相应的.mtail结尾的配置文件，就可以启动了。

```shell
systemctl daemon-reload
systemctl start mtail
# 查看导出的指标中是否包含解析规则中定义的指标
curl localhost:3903/metrics
```

### 编程

mtail还支持对解析出的指标进行相应的编程和计算。详情请查看：[mtail 编程](https://github.com/google/mtail/blob/master/docs/Programming-Guide.md)。

## 调试

在首次部署时，需要对日志配置相应的解析规则，可能会出现无法解析日志，原因是规则不匹配。mtail在每次启动时，都会在TMP目录下生成 `mtail.INFO/mtail.WARNING/mtail.ERROR`相关日志（在未指定mtail本身日志目录的情况下），注意查看报错。

## 关于生成指标

如果时序数据库使用的是Prometheus，在生成日志指标数据时，指标的label中包含类似uri路径时，需要注意一下。因为每一个uri中参数或者id不一样，导致生成了很多指标数据。Prometheus建议不应该将exporter导出数据基数过大，避免影响exporter的性能和统计，因为每一个不同的标签都会生成一条记录。一个服务的API数量是有限的，可以将API请求路径中的动态参数换成指定的变量，如 `/api/v1/user/123`换成`/api/v1/user/:id`，这样多个`/api/v1/user/:id`就只会成为一个统计和。目前mtail还不支持这个功能，[相关issue](https://github.com/google/mtail/issues/291)。不过另外的exporter似乎可以，如[grok_exporter](https://github.com/fstab/grok_exporter)可以使用gsub功能来完成。
