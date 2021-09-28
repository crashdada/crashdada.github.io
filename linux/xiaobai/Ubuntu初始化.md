# Ubuntu服务器初始化

## 设置 SSH 公钥登录

本地执行：

```bash
ssh-keygen -b 4096 # 本地没有 SSH 密钥对时执行
brew install ssh-copy-id # 本地没有 ssh-copy-id 时执行
ssh-copy-id remote_username@remote_host
```

## 安装更新

如果出现提示，选择「keep the local version currently installed」。

```bash
sudo apt update
sudo apt -y upgrade
```

## 设置主机名称

```bash
HOSTNAME=your-hostname
sudo hostnamectl set-hostname $HOSTNAME
# echo "127.0.0.1 $HOSTNAME" | sudo tee -a /etc/hosts
```

## 设置时区

```bash
sudo timedatectl set-timezone UTC
sudo timedatectl set-local-rtc 0
sudo timedatectl set-ntp 1
```

## 创建用户

```bash
adduser username
adduser username sudo
```

## 配置 SSH

```shell
 vi /etc/ssh/sshd_config
```

### 禁止使用 root 用户登录

```text
PermitRootLogin no
```

### 禁止使用密码登录

```text
PasswordAuthentication no
```

### 限制只监听 IPv4

SSH 默认同时监听 IPv4 和 IPv6。

```bash
echo 'AddressFamily inet' | sudo tee -a /etc/ssh/sshd_config # inet 改为 inet6 即为只监听 IPv6
```

### 重启 SSH 服务

```bash
systemctl restart sshd
```