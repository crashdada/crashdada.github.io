# CentOS 7 初始化配置

## 前期准备

- 分配域名解析
- 分配 root 密码
- 分配 ssh 端口
- 分配项目端口
- 配置控制台防火墙端口

## 流程大纲

1. 登陆配置
2. 软件配置
3. 卸载监控
4. 重启快照

!> 国内服务器一般默认禁用防火墙，由控制台管理。国外服务器视情况可能由系统防火墙管理。

## 初始化流程

### 登陆配置

#### 服务器登陆

如果服务器使用非 root 账户登陆,登陆后先切换至 root 账户.

```shell
sudo -i
# 如果无 root 密码则执行以下命令修改或创建密码
passwd
```

#### 证书配置

!> **`本地生成 ssh key`**

```shell
ssh-keygen -t rsa -C "youremail@example.com"
```

复制`id_rsa.pub`内容到服务器`~/.ssh/authorized_keys`

```shell
mkdir ~/.ssh
vi ~/.ssh/authorized_keys
```

新创建的`authorized_keys`文件需配置权限

```shell
chmod 600 ~/.ssh/authorized_keys
```

甲骨文用户注意:默认`authorized_keys`中会有判断是否为`opc`用户登陆,需要删除`ssh-rsa`之前的判断语句.

#### 配置其他登陆项

```shell
systemctl stop firewalld
systemctl disable firewalld
# 关闭防火墙或放行修改后的 SSH 端口
vi /etc/ssh/sshd_config
# 修改登陆配置文件
Port 22222
# 修改 SSH 登陆端口号,请确保上述已关闭防火墙,并在服务商控制面板开放端口.
PermitRootLogin yes
# 允许 root 账户直接登陆
PasswordAuthentication yes
# 开启密码验证.如使用证书登陆,则修改为 no 确保安全性.
ClientAliveInterval 30
MaxSessions 100
# 重启 sshd 生效
systemctl restart sshd
```

#### 禁用其他账户登陆

```shell
vi /etc/passwd
# 注释不允许登陆的账号
```

#### 修改主机名

修改命令

```shell
hostnamectl set-hostname xxxxx
```

重启不生效

!> 部分服务商系统修改主机名重启后不生效,相关解决办法如下:

```shell
# 甲骨文
vi /etc/oci-hostname.conf
# 将 PRESERVE_HOSTINFO=0 修改为 1

# 其他
vi /etc/hostname
vi /etc/hosts
# 手动修改 hostname 及 hosts
vi /etc/sysconfig/network
# 同时查看 network 是否定义 HOSTNAME
```

### 软件配置

#### yum repo

```shell
yum repolist
yum repolist all
# 查看 yum 源
yum -y install epel-release
# 如无 epel/x86_64 推荐先安装 epel 源
yum clean all
yum makecache
# 清除并重建缓存
yum -y update
# 更新
# 如需使用 epel 源国内阿里云镜像加速,执行以下命令:
mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup
mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.backup
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum clean all
yum makecache
```

甲骨文 ARM Oracle Linux

```shell
cat >/etc/yum.repos.d/centos-extras.repo << 'EOF'
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://mirrorlist.centos.org/?release=7&arch=$basearch&repo=extras&infra=$infra
gpgcheck=0
EOF
```

#### yum install

```shell
yum -y update
yum -y install wget zip unzip tar git screen vnstat telnet nethogs net-tools iptables-services bash-completion bind-utils lsof
```

#### iptables

```shell
yum -y install iptables-services
# 安装 iptables iptables-services

systemctl stop firewalld
systemctl disable firewalld
# 禁用 firewall

systemctl start iptables
systemctl status iptables
systemctl enable iptables
# 启用 iptables

iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -F 
service iptables save
# 设置默认规则并清空所有规则,保存设置.
```

#### ServerStatus

```shell
wget -N --no-check-certificate https://raw.githubusercontent.com/stilleshan/ServerStatus/master/status.sh && chmod +x status.sh && ./status.sh c
```

#### swap

```shell
free -m
# 检查 swap 状态
wget https://files.ioiox.com/projects/swap/swap.sh && chmod +x swap.sh && ./swap.sh
cat /etc/fstab
# 检查 swap 挂载
```

#### vnStat

!>  安装完毕执行`ifconfig`查看本机网卡名,并修改配置文件.
ARM 架构访问 [vergoh/vnstat](https://github.com/vergoh/vnstat/blob/master/INSTALL.md) 编译安装.

安装

```shell
yum -y install vnstat
systemctl start vnstat
systemctl enable vnstat
```

命令

```shell
vnstat -l
# Show Statistics for a Real Time

vnstat -h
# Show Statistics for a Hour

vnstat -d
# Show Statistics for a day

vnstat -w
# Show Statistics for a week

vnstat -m
# Show Statistics for a Month

vnstat --help
# For more available options you can use the --help
```

配置

```shell
# 配置文件
vi /etc/vnstat.conf

# 数据库目录
cd /var/lib/vnstat/
```

#### NTP

```shell
tzselect
# 选择时区
mv /etc/localtime /etc/localtime.bak
# 备份 localtime
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# 创建时区软链接,再次 date 检查时区.
yum -y install ntpdate
# 安装 ntpdate
ntpdate time.apple.com
# 同步时间
# 备选 ntp 服务器
time.apple.com
cn.pool.ntp.org
ntp.aliyun.com
ntp1.aliyun.com
time1.cloud.tencent.com
# crontab 定时更新
55 23 * * * /usr/sbin/ntpdate time.apple.com >/dev/null 2>&1
# 如遇到 the NTP socket is in use, exiting 问题,尝试先停止 ntpd 服务后在次执行同步命令
systemctl stop ntpd
```

#### sendmail

```shell
# 安装 sendmail
yum -y install sendmail*
# 安装 mailx
yum -y install mailx
```

发送测试

```shell
echo "this is my test mail" | mail -s 'mail test' xxx@xxx.com
# 文本测试
mail -s 'mail test' xxx@xxx.com < xxx.txt
# 文件测试
```

配置 smtp

```shell
vi /etc/mail.rc
# 添加以下配置,注意 smtp 可能因为云服务商之间限制无法发送,尝试加上 80,465 端口,再次测试.
set from=xxx@xxx.com
set smtp=smtp.xxx.com
set smtp-auth-user=username
set smtp-auth-password=password
set smtp-auth=login
```

配置 root 登陆邮件提醒

```shell
vi ~/.bash_profile
```

添加以下模版

```shell
IP="$(echo $SSH_CONNECTION | cut -d " " -f 1)"
HOSTNAME=$(hostname)
NOW=$(date +"%e %b %Y, %a %r")
echo 'Someone from '$IP' logged into '$HOSTNAME' on '$NOW'.' | mail -s 'SSH Login Notification' YOUR_EMAIL_ADDRESS
# YOUR_EMAIL_ADDRESS 修改为接收通知的邮箱地址
```

关闭`您在 /var/spool/mail/root 中有新邮件`提示

```shell
echo "unset MAILCHECK">> /etc/profile
```

#### BBR

!> 甲骨文 X86 和 ARM 架构 BBR 或 BBRPlus 参考 [Link1](https://www.ioiox.com/archives/63.html) [Link2](https://www.ioiox.com/archives/134.html)

```shell
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```

#### docker

!> 甲骨文 ARM 架构 Linux 安装 docker 及 docker-compose 参考 [Link1](https://www.ioiox.com/archives/134.html) [Link2](https://www.ioiox.com/archives/135.html)

```shell
# 安装 docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
# 国内镜像
sudo sh get-docker.sh --mirror Aliyun
# 启动
sudo systemctl start docker
sudo docker version
sudo systemctl enable docker
# docker-compose
curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose version
```

### 卸载监控

#### 卸载阿里云盾安骑士

官方卸载脚本

```shell
wget http://update.aegis.aliyun.com/download/uninstall.sh && chmod +x uninstall.sh && ./uninstall.sh
wget http://update.aegis.aliyun.com/download/quartz_uninstall.sh && chmod +x quartz_uninstall.sh && ./quartz_uninstall.sh
```

删除阿里云盾文件残留

```shell
pkill aliyun-service
rm -rf /etc/init.d/agentwatch /usr/sbin/aliyun-service
rm -rf /usr/local/aegis*
```

屏蔽阿里云盾IP

```shell
iptables -I INPUT -s 140.205.201.0/28 -j DROP
iptables -I INPUT -s 140.205.201.16/29 -j DROP
iptables -I INPUT -s 140.205.201.32/28 -j DROP
iptables -I INPUT -s 140.205.225.192/29 -j DROP
iptables -I INPUT -s 140.205.225.200/30 -j DROP
iptables -I INPUT -s 140.205.225.184/29 -j DROP
iptables -I INPUT -s 140.205.225.183/32 -j DROP
iptables -I INPUT -s 140.205.225.206/32 -j DROP
iptables -I INPUT -s 140.205.225.205/32 -j DROP
iptables -I INPUT -s 140.205.225.195/32 -j DROP
iptables -I INPUT -s 140.205.225.204/32 -j DROP
```

一键执行

```shell
iptables -I INPUT -s 140.205.201.0/28 -j DROP && \
iptables -I INPUT -s 140.205.201.16/29 -j DROP && \
iptables -I INPUT -s 140.205.201.32/28 -j DROP && \
iptables -I INPUT -s 140.205.225.192/29 -j DROP && \
iptables -I INPUT -s 140.205.225.200/30 -j DROP && \
iptables -I INPUT -s 140.205.225.184/29 -j DROP && \
iptables -I INPUT -s 140.205.225.183/32 -j DROP && \
iptables -I INPUT -s 140.205.225.206/32 -j DROP && \
iptables -I INPUT -s 140.205.225.205/32 -j DROP && \
iptables -I INPUT -s 140.205.225.195/32 -j DROP && \
iptables -I INPUT -s 140.205.225.204/32 -j DROP && \
service iptables save
```

检查阿里云盾是否卸载干净

```shell
ps -aux | grep -E 'aliyun|AliYunDun'
# 检查进程是否还有 AliYunDun、aliyun-service和AliYunDunUpdate
```

#### 卸载腾讯云监控程序

```shell
/usr/local/qcloud/stargate/admin/uninstall.sh
/usr/local/qcloud/YunJing/uninst.sh
/usr/local/qcloud/monitor/barad/admin/uninstall.sh
systemctl stop tat_agent
systemctl disable tat_agent
rm -f /etc/systemd/system/tat_agent.service
# 检查进程为空
ps -A | grep agent
# 同时删除 crontab 中的定时任务计划
crontab -e
```

#### 卸载甲骨文监控程序

```shell
systemctl stop oracle-cloud-agent
systemctl disable oracle-cloud-agent
systemctl stop oracle-cloud-agent-updater
systemctl disable oracle-cloud-agent-updater
```

RPC 111 端口

```shell
systemctl stop rpcbind
systemctl stop rpcbind.socket
systemctl disable rpcbind
systemctl disable rpcbind.socket 
```

### 重启快照

```shell
reboot
```

#### 再次检查

- uname -r
- date
- free -m
- serverstatus
- BBR
- iptables -L
- crontab -l
- netstat -ntlp
- root 目录清理脚本
- 云服务商监控程序

#### 清除记录重启创建快照

手动清除 root 目录下无用文件,清除历史登陆记录,清除历史命令,重启并创建快照.

```shell
echo > /var/log/wtmp && history -c && history -w && reboot
# 清除并重启服务器
echo > /var/log/wtmp && history -c && history -w && exit
# 清除并退出登录
```

