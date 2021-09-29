# Ubuntu 20.04快速开启TCP BBR

`Linux Kernel` 内核升级到 `4.9` 及以上版本可以实现 `BBR` 加速，由于`Ubuntu 20.04` 默认的内核就是 `5.4` 版本的内核，并已经默认编译了 `TCP BBR` 模块，所以可以直接通过参数开启。

新的 `TCP` 拥塞控制算法 `BBR` (Bottleneck Bandwidth and RTT) 可以让服务器的带宽尽量跑慢，并且尽量不要有排队的情况，让网络服务更佳稳定和高效。

## 开启操作

**修改系统变量**

```shell
echo net.core.default_qdisc=fq >> /etc/sysctl.conf
echo net.ipv4.tcp_congestion_control=bbr >> /etc/sysctl.conf
```

**保存生效**

```shell
sudo sysctl -p
```

## 检验是否开启

**执行**

```shell
sysctl net.ipv4.tcp_available_congestion_control
```

返回类似

```text
net.ipv4.tcp_available_congestion_control = reno cubic bbr
```

 执行

```shell
lsmod | grep bbr
```

返回类似

```
tcp_bbr                20480  1
```

**成功开启**

## **建议重启系统**

**重启后执行**

```shell
sysctl net.ipv4.tcp_congestion_control net.ipv4.tcp_congestion_control = bbr
```