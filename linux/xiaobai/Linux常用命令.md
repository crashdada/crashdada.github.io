# Linux常用命令

## 日常命令

```shell
cd                                          // 进入某目录 举例：cd /home/wwwroot
ls                                          // 查看当前目录文件
lsb_release -a                              // 查看系统版本
uname -r                                    // 查看内核
uname -a                                    // 查看内核/操作系统/CPU信息
head -n 1 /etc/issue                        // 查看操作系统版本
cat /proc/cpuinfo                           // 查看CPU信息
hostname                                    // 查看计算机名
lspci -tv                                   // 列出所有PCI设备
lsusb -tv                                   // 列出所有USB设备
lsmod                                       // 列出加载的内核模块
env                                         // 查看环境变量
cp                                          // 复制
rm                                          // 删除

// 修改主机名
hostname                                    // 查看主机名
hostname SunPma                             // 临时修改主机名为SunPma
vi /etc/hostname                            // 永久修改主机名（重启后生效）

// 安装VI编辑器
apt-get install vim
i                                           // 开始编辑
Esc                                         // 完成编辑
:wq                                         // 保存退出
:q！                                        // 退出，不保存

// Nohup后台运行命令
nohup top &                                 // 挂载后台运行 例：将top进程挂载到后台运行
ps -ef                                      // 查看进程
kill -6 8585                                // 结束进程    例：结束8585进程

// 移动命令
// 说明：把abc.tar.gz移动到backup目录
mv /home/wwwroot/abc.tar.gz /home/backup

// 删除命令
rm -rf /var/log/httpd/access
// 将会删除/var/log/httpd/access目录以及其下所有文件、文件夹
// -r 就是向下递归，不管有多少级目录，一并删除
// -f 就是直接强行删除，不作任何提示的意思
// 如文件夹命有空格，需加引号
// 如：rm -rf /home/box123/downloads/'Naughty America SiteRip 4K Part13-sweety'
rm -rf /var/log/httpd/access/*
// 将会删除/var/log/httpd/access目录下所有文件、文件夹，但不删除access文件夹本身
// 删除不可恢复，请谨慎使用；

// SSD硬盘测试
// 安装hdparm
install hdparm -y
// 找出对应磁盘的驱动号
fdisk -l
// 利用hdparm在指定的硬盘上测试硬盘的读写速度
hdparm -t /dev/hda

// 查看硬盘使用时间
apt-get install smartmontools
smartctl -A /dev/sda
smartctl -s on --all /dev/sda | grep Power_On_Hours
// Power_On_Hours 后面的这个就是通电时间, 单位为小时
// RAID阵列
smartctl -a /dev/sg0
smartctl -a /dev/sg1
smartctl -a /dev/sg2
// 或者
smartctl -s on --all /dev/sg0 | grep Power_On_Hours
smartctl -s on --all /dev/sg1 | grep Power_On_Hours
smartctl -s on --all /dev/sg2 | grep Power_On_Hours

// IPV6测试
// VPS
ping6 ipv6.baidu.com
ping6 ipv6.google.com
// PC
ping -6 240c::6666
ping -6 2001:da8::666
ping -6 2001:da8:8000:1:202:120:2:100

// 修改系统预留硬盘空间
// 查看当前硬盘空间情况
df -h
// 调整/dev/sda1的预留空间，只保留0.1%的空间
tune2fs -m 0.1 /dev/sda1

// 修改用户组
// -R命令为递归，含里面的子目录和全部文件
chown sunpma:sunpma /home/sunpma/Aria2
chown -R sunpma:sunpma /home/sunpma/Aria2

// 修改权限
// -R命令为递归，含里面的子目录和全部文件
chmod 777 /home/sunpma/Aria2
chmod -R 777 /home/sunpma/Aria2

// 关闭邮件通知
// 第一步：关闭提示
echo "unset MAILCHECK">> /etc/profile
source /etc/profile
// 第二步：查看
ls -lth /var/spool/mail/
// 第三步：清空
cat /dev/null > /var/spool/mail/root

// 释放内存
echo 1 > /proc/sys/vm/drop_caches
echo 2 > /proc/sys/vm/drop_caches
echo 3 > /proc/sys/vm/drop_caches
// 说明
// echo 1 >清除pagecache
// echo 2 >清除回收slab分配器中的对象
// echo 3 >清除pagecache和slab分配器中的缓存对象

// 复制指定后缀文件
// 说明：复制/home/sunpma/目录中的所有jpg后缀文件到/home/ceshi/目录下
find /home/sunpma/ -name "*.jpg" -exec cp {} /home/ceshi/ \;

// 查看系统运行时间及实时负载
uptime

// 关闭宝塔账户登录
rm -rf /www/server/panel/data/bind.pl

// 添加hosts条目
echo '127.0.0.1 baidu.com' >>/etc/hosts
// 例：国内机器修改hosts连接GitHub
echo "140.82.114.4 github.com" >> /etc/hosts
echo "185.199.109.153 github.io" >> /etc/hosts
echo "185.199.108.133 raw.githubusercontent.com" >> /etc/hosts
echo "199.232.5.194 github.global.ssl.fastly.net" >> /etc/hosts
// Windows PC
C:\Windows\System32\drivers\etc

// 磁盘I/O测试
dd if=/dev/zero of=benchtest_30127 bs=64k count=16k conv=fdatasync
dd if=/dev/zero of=/tmp/test.img bs=1M count=1024 oflag=dsync && rm -rf /tmp/test.img

// 复制目录下所有文件到另一目录
cp -rf /home/sunpma/* /home/sunpma/mm/

// 删除目录下所有文件
rm -rf /home/sunpma/*
```

## 进程相关

```shell
// 查看所有进程
ps -ef

// 查看指定程序名的所有进程
ps -ef | grep 程序名称

// 正常结束进程
kill 进程PID
killall 进程名称

// 强制结束进程
kill -9 进程PID
killall -9 进程名称
```

## 关闭防火墙

```shell
// Debian/Ubuntu 关闭防火墙
apt-get remove ufw 
iptables -P INPUT ACCEPT 
iptables -P OUTPUT ACCEPT 
iptables -F 

// CentOS 关闭防火墙
systemctl stop firewalld.service 
systemctl disable firewalld.service 
yum install iptables iptables-services 
iptables -P INPUT ACCEPT 
iptables -P OUTPUT ACCEPT 
iptables -F
```

## 临时文件目录

```shell
// 使用Python创建一个临时文件目录
// 进入需要挂载的目录
cd /home/sunpma
// Python
python -m SimpleHTTPServer 8000
// Python3
python3 -m http.server 8000
```

## 禁用IPv6

```shell
// 编辑文件
vi /etc/sysctl.d/99-sysctl.conf

// 在最后添加如下代码
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

// 使其生效
sysctl -p
```

## 放行端口

```shell
// Debian/Ubuntu
iptables -I INPUT -p tcp --dport 8888 -j ACCEPT
iptables-save
// 安装iptables-persistent使规则持续生效
apt-get install iptables-persistent
netfilter-persistent save
netfilter-persistent reload

// Centos
firewall-cmd --zone=public --add-port=8888/tcp --permanent
firewall-cmd --reload

// 查看防火墙规则
iptables -L
```

## 卸载阿里云盾监控

```shell
// 卸载云盾监控
wget http://update.aegis.aliyun.com/download/uninstall.sh
sh uninstall.sh
wget http://update.aegis.aliyun.com/download/quartz_uninstall.sh
sh quartz_uninstall.sh

// 删除目录残留
pkill aliyun-service
rm -fr /etc/init.d/agentwatch /usr/sbin/aliyun-service
rm -rf /usr/local/aegis*
```

## WGET下载命令

```shell
// 安装wget

apt-get update
apt-get install wget
wget --version
// wget命令 下载单个文件
wget https://xx.com

// wget命令 下载并重命名
wget -O xx.zip https://xx.com

// wget命令 下载到指定目录
wget -P /home/abc https://xx.com

// wget命令 下载到指定目录并重名
wget -O /home/xx.zip https://xx.com

// wget命令 限速下载
wget --limit-rate=500k https://xx.com

// wget命令 断点续传
wget -c https://xx.com

// wget命令 后台下载
wget -b https://xx.com

// wget命令 伪装代理名称下载
wget --user-agent="Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.16 (KHTML, like Gecko) Chrome/10.0.648.204 Safari/534.16" https://xx.com/xx

// wget命令 批次下载多个文件
wget -i filelist.txt #我们制作一个filelist.txt文件，然后文档中放置多个文件需要下载。一行一个链接文件

// wget命令 限制送文件大小下载
wget -Q5m -i filelist.txt
```

## TOP命令

```shell
// top       # 实时显示进程状态
// ps -ef    # 查看所有进程
// 例：
Tasks: 29 total, 1 running, 28 sleeping, 0 stopped, 0 zombie

Cpu(s): 0.3% us, 1.0% sy, 0.0% ni, 98.7% id, 0.0% wa, 0.0% hi, 0.0% si

-------------------------------- 分割线 -------------------------------

Tasks: 29 total   # 进程总数
1 running         # 正在运行的进程数
28 sleeping       # 睡眠的进程数
0 stopped         # 停止的进程数
0 zombie          # 僵尸进程数
Cpu(s):
0.3% us           # 用户空间占用CPU百分比
1.0% sy           # 内核空间占用CPU百分比
0.0% ni           # 用户进程空间内改变过优先级的进程占用CPU百分比
98.7% id          # 空闲CPU百分比
0.0% wa           # 等待输入输出的CPU时间百分比
0.0% hi
0.0% si
0.0% wa           # wa百分比可以大致的体现出当前的磁盘io请求是否频繁；
                  # 如果wa的数量比较大，说明等待输入输出的的io比较多。
```

## tar.gz解压缩工具

```shell
// 解压tar包
tar -xvf file.tar
// 解压tar.gz
tar -xzvf file.tar.gz
// 解压tar.bz2
tar -xjvf file.tar.bz2
// 解压tar.Z
tar -xZvf file.tar.Z
// 解压rar
unrar e file.rar
// 解压zip
unzip file.zip

// 将/home/wwwroot压缩
tar zcvf test.tar.gz /home/wwwroot

// 查看文件真正属性
file file.tar.gz
```

## UNZIP解压缩工具

```shell
// 获取unzip源码
wget http://downloads.sourceforge.net/infozip/unzip552.tar.gz
// 解压
tar zxvf unzip552.tar.gz
// 进入目录
cd unzip-5.52/
// 将Makefile从unix子目录复制到当前目录
cp unix/Makefile ./
// 安装
make generic
make install

// 当前目录下解压缩
unzip test.zip
// 查看压缩文件目录
unzip -v test.zip
// 在指定目录下解压缩
unzip test.zip tmp/
```

## 系统信息

```shell
arch                   // 显示机器的处理器架构
uname -m               // 显示机器的处理器架构
uname -r               // 显示正在使用的内核版本 
dmidecode -q           // 显示硬件系统部件 - (SMBIOS / DMI) 
hdparm -i /dev/hda     // 罗列一个磁盘的架构特性 
hdparm -tT /dev/sda    // 在磁盘上执行测试性读取操作 
cat /proc/cpuinfo      // 显示CPU info的信息 
cat /proc/interrupts   // 显示中断 
cat /proc/meminfo      // 校验内存使用 
cat /proc/swaps        // 显示哪些swap被使用 
cat /proc/version      // 显示内核的版本 
cat /proc/net/dev      // 显示网络适配器及统计 
cat /proc/mounts       // 显示已加载的文件系统 
lspci -tv              // 罗列 PCI 设备 
lsusb -tv              // 显示 USB 设备 
date                   // 显示系统日期 
cal 2007               // 显示2007年的日历表 
date 041217002007.00   // 设置日期和时间 - 月日时分年.秒 
clock -w               // 将时间修改保存到 BIOS
```

## 资源查看

```shell
free -m                         // 查看内存使用量和交换区使用量
df -h                           // 查看各分区使用情况
du -sh                          // 查看指定目录的大小
grep MemTotal /proc/meminfo     // 查看内存总量
grep MemFree /proc/meminfo      // 查看空闲内存量
uptime                          // 查看系统运行时间、用户数、负载
cat /proc/loadavg               // 查看系统负载
```

## 磁盘和分区

```shell
df -h                  // 查看各分区使用情况
mount | column -t      // 查看挂接的分区状态
fdisk -l               // 查看所有分区
swapon -s              // 查看所有交换分区
hdparm -i /dev/hda     // 查看磁盘参数(仅适用于IDE设备)
dmesg | grep IDE       // 查看启动时IDE设备检测状况
```

## 网络

```shell
ifconfig               // 查看所有网络接口的属性
iptables -L            // 查看防火墙设置
route -n               // 查看路由表
netstat -lntp          // 查看所有监听端口
netstat -antp          // 查看所有已经建立的连接
netstat -s             // 查看网络统计信息

// Ubuntu常用的网络指令
/etc/init.d/networking start          // 启动网络服务
/etc/init.d/networking stop           // 关闭网络服务
/etc/init.d/networking restart        // 重启网络服务
/etc/init.d/networking force-reload   // 强制重载网络配置
// 指定重载网卡
apt install ifupdown                  // 安装ifupdown
ifup ens3                             // 重启ens3网卡
```

## 用户

```shell
su sunpma                  // 切换用户
w                          // 查看活动用户
id root                    // 查看指定用户信息
last                       // 查看用户登录日志
cut -d: -f1 /etc/passwd    // 查看系统所有用户
cut -d: -f1 /etc/group     // 查看系统所有组
crontab -l                 // 查看当前用户的计划任务
useradd root               // 创建用户
passwd root                // 设置密码
userdel -r root            // 删除用户
```

## 服务

```shell
chkconfig list            // 列出所有系统服务
chkconfig list | grep on  // 列出所有启动的系统服务程序
rpm -qa                   // 查看所有安装的软件包
```

## 文件搜索

```shell
find / -name file1 从 '/' 开始进入根文件系统搜索文件和目录 
find / -user user1 搜索属于用户 'user1' 的文件和目录 
find /home/user1 -name \*.bin 在目录 '/ home/user1' 中搜索带有'.bin' 结尾的文件 
find /usr/bin -type f -atime +100 搜索在过去100天内未被使用过的执行文件 
find /usr/bin -type f -mtime -10 搜索在10天内被创建或者修改过的文件 
find / -name \*.rpm -exec chmod 755 '{}' \; 搜索以 '.rpm' 结尾的文件并定义其权限 
find / -xdev -name \*.rpm 搜索以 '.rpm' 结尾的文件，忽略光驱、捷盘等可移动设备 
locate \*.ps 寻找以 '.ps' 结尾的文件 - 先运行 'updatedb' 命令 
whereis halt 显示一个二进制文件、源码或man的位置 
which halt 显示一个二进制文件或可执行文件的完整路径
```

