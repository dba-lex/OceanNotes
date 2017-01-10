

## 使用tc命令控制网络延迟

- 该命令将 eth0 网卡的传输设置为延迟 100ms ± 10ms (90 ~ 110 ms 之间的任意值)发送。还可以更进一步加强这种波动的随机性:

```
# tc qdisc add dev eth0 root netem delay 100ms 10ms
```

- 该命令将 eth0 网卡的传输设置为 100ms ,同时,大约有 30% 的包会延迟 ± 10ms 发送。

```
# tc qdisc add dev eth0 root netem delay 100ms 10ms 30%
```

- 删除tc规则

```
# tc qdisc del dev eth0 root
```

- 查看并显示 eth0 网卡的相关传输配置

```
# tc qdisc show dev eth0
```
