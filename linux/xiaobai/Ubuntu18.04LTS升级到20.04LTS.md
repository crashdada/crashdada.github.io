# Ubuntu18.04LTS升级到20.04LTS

## 正式开始

整个过程分成了**四步**，时间取决于你的网络连接速度（下载大概1.6G数据），和你的电脑性能，毕竟是系统整体升级。

大部分的过程是无需太多干预的，只是偶尔有些选项需要你做个决定，敲个“Y”或者“N”之类的。

## 一、备份

老生常谈了，备份的重要性，可以在关键时刻救你一命，所以把你认为重要的文件，直接拷贝出来吧，那个目录你觉得有必要就直接拷贝，升级完需要的话直接拷贝回来就可以了，比如我们的HA目录（/usr/share/hassio）等等。

```shell
ssh username@serverIP
##再输入密码就远程连接到服务器了

lsb_release -a
```

输出的内容大概是这样，记下来。

```text
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.4 LTS
Release:	18.04
Codename:	bionic
```

然后

```shell
uname -mrs
```

输出的内容大概是这样

```text
Linux 4.15.0-96-generic x86_64
```

## 二、更新所有安装包

```shell
sudo apt update
sudo apt list --upgradable
sudo apt upgrade
```

执行完成后，重启电脑

```shell
sudo reboot
```

## 三、清理系统

趁着升级，把系统中不必要的软件包全部都清理一下

```shell
sudo apt --purge autoremove
```

升级一下系统升级管理工具

```shell
sudo apt install update-manager-core
```

## 四、**升级！！！**

前面的功夫都做完之后，就到了正式升级的时候

```shell
sudo do-release-upgrade
```

如果输出的是下面的内容：

```text
Checking for a new Ubuntu release There is no development version of an LTS available. To upgrade to the latest non-LTS develoment release set Prompt=normal in /etc/update-manager/release-upgrades.
```

那么再次执行以下命令：

```shell
sudo do-release-upgrade -d
```

后续基本上就是选择yes还是no的问题，提示默认（Default=n）的，你就选n。然后就是继续一屏一屏的字母在跑了。这就是系统在正式升级更新啦。你能做的就是——等等等灯

## **大功告成**

