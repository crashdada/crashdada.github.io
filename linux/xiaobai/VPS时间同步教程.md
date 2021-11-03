#  VPS时间同步教程

## 简介

在配置V2ray的时候，因VPS时间和本地客户端时间不同步，无法正常使用。于是今天就分享个同步VPS时间的教程。

## 设置时区为北京时间

一般国外的VPS的镜像都是默认的国外时区，使用起来不是很方便。可以把它修改成北京时间，就会方便很多。代码：

```shell
rm -rf /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## NTP同步时间协议

众所周知，NTP协议是网络时间同步协议，有了它，我们可以很轻松的同步本地时间与互联网时间。VPS上也可以使用NTP来同步网络。首先安装必要的软件包：

### Ubuntu/Debian系统

```shell
apt-get update
apt-get install ntp ntpdate -y
```

### CentOS/RHEL系统

```shell
yum install ntp ntpdate -y
```

接下来我们需要先停止NTP服务器，再更新时间。

```shell
service ntpd stop                 #停止ntp服务
ntpdate cn.pool.ntp.org           #同步ntp时间
service ntpd start                #启动ntp服务
```

## 同步硬件时钟

linux系统时钟有两个，一个是硬件时钟，即BIOS时间，就是我们进行CMOS设置时看到的时间，另一个是系统时钟，是linux系统Kernel时间。当Linux启动时，系统Kernel会去读取硬件时钟的设置，然后系统时钟就会独立于硬件运作。我们一般改动都是系统时间，要解决问题根源的方法是修改硬件时间。

```shell
hwclock --systohc
```

## 查看时间

查看系统时间命令

```shell
date
```


查看硬件时间命令

```shell
clock -r
```

## 完工

执行完成后，VPS上就是相对精确的时间设置了。很多依赖于系统时间的应用程序也就能正常工作了。