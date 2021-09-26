# OpenConnect VPN 自签证书配置

## 目录规划

本文是针对 OpenConnect VPN 的证书签发,吊销,取消吊销和管理进行规划整理,方便后期查阅和复用.

### 证书主目录

主目录路径: `/etc/ocserv/ssl` 主要包含:

- CA 根证书模版 `ca.tmp`
- CA 根证书私钥 `ca.key.pem`
- CA 根证书 `ca.cert.pem`
- Diffie-Hellman 密钥 `dh.pem`

### 用户证书目录

用户证书目录路径: `/etc/ocserv/ssl/user` 主要包含:

- 用户证书模版 `stille.tmp`
- 用户证书私钥 `stille.key.pem`
- 用户证书 `stille.cert.pem`
- 用户`AnyConnect.p12`证书 `sitlle.AnyConnect.p12`

### 吊销证书目录

吊销证书目录: `/etc/ocserv/ssl/revoked` 主要包含:

- 吊销模版 `crl.tmpl`
- 吊销文件 `revoked.pem`
- 吊销列表文件 `crl.pem`

## 自签证书

### 创建目录

创建`ssl`目录,以存放`ca 根证书`相关文件.

```shell
mkdir -p /etc/ocserv/ssl
```

创建`user`目录,以存放`用户证书`相关文件.

```shell
mkdir -p /etc/ocserv/ssl/user
```

创建`revoked`目录,以存放`吊销证书`相关文件.

```shell
mkdir -p /etc/ocserv/ssl/revoked
```

### 自签 CA 根证书

#### 创建 CA 根证书模版

```shell
cat >/etc/ocserv/ssl/ca.tmp<<EOF
cn = "jaymz CA"
organization = "jaymz"
serial = 1
expiration_days = 3650
ca
signing_key
cert_signing_key
crl_signing_key
EOF
```

#### 生成 CA 根证书私钥

```shell
openssl genrsa -out /etc/ocserv/ssl/ca.key.pem 2048
```

#### 根据 CA 根证书私钥和 CA 根证书模版生成 CA 根证书

```shell
certtool --generate-self-signed --hash SHA256 --load-privkey /etc/ocserv/ssl/ca.key.pem --template /etc/ocserv/ssl/ca.tmp --outfile /etc/ocserv/ssl/ca.cert.pem
```

#### 生成 Diffie-Hellman 密钥

```shell
certtool --generate-dh-params --outfile /etc/ocserv/ssl/dh.pem
```

### 自签用户证书

#### 创建用户证书模版

```shell
cat >/etc/ocserv/ssl/user/jun.tmp<<EOF
cn = "jun"
unit = "jaymz"
expiration_days = 3650
signing_key
tls_www_client
EOF
```

#### 生成用户证书私钥

```shell
openssl genrsa -out /etc/ocserv/ssl/user/jun.key.pem 2048
```

#### 根据用户证书私钥, CA 根证书, CA 根证书私钥,用户证书模版生成用户证书

```shell
certtool --generate-certificate --hash SHA256 --load-privkey /etc/ocserv/ssl/user/jun.key.pem --load-ca-certificate /etc/ocserv/ssl/ca.cert.pem --load-ca-privkey /etc/ocserv/ssl/ca.key.pem --template /etc/ocserv/ssl/user/jun.tmp --outfile /etc/ocserv/ssl/user/jun.cert.pem
```

#### 证书链补全

```shell
cat /etc/ocserv/ssl/ca.cert.pem >>/etc/ocserv/ssl/user/jun.cert.pem
```

#### 生成 .p12 证书文件

!> 注意修改最后的密码XXX

```shell
openssl pkcs12 -export -inkey /etc/ocserv/ssl/user/jun.key.pem -in /etc/ocserv/ssl/user/jun.cert.pem -name "jun" -certfile /etc/ocserv/ssl/ca.cert.pem -caname "jaymz CA" -out /etc/ocserv/ssl/user/jun.AnyConnect.p12 -passout pass:XXX
```

至此CA 根证书和用户证书已成功生成.将jun.AnyConnect.p12证书拷贝至电脑以备导入使用.
.p12 用户证书所在路径为: `/etc/ocserv/ssl/user/jun.AnyConnect.p12`

### 吊销证书

#### 创建吊销模版

```shell
cat <<EOF >/etc/ocserv/ssl/revoked/crl.tmpl
crl_next_update = 365
crl_number = 1
EOF
```

#### 创建空的吊销文件

```shell
touch /etc/ocserv/ssl/revoked/revoked.pem
```

#### 根据 CA 根证书私钥, CA 根证书,吊销模版生成空的吊销列表文件

```shell
certtool --generate-crl --load-ca-privkey /etc/ocserv/ssl/ca.key.pem --load-ca-certificate /etc/ocserv/ssl/ca.cert.pem --template /etc/ocserv/ssl/revoked/crl.tmpl --outfile /etc/ocserv/ssl/revoked/crl.pem
```

#### 将需要吊销的用户证书写入吊销文件

```shell
cat /etc/ocserv/ssl/user/jun.cert.pem >> /etc/ocserv/ssl/revoked/revoked.pem
```

#### 根据 CA 根证书私钥, CA 根证书,吊销文件,吊销模版写入吊销列表文件

```shell
certtool --generate-crl --load-ca-privkey /etc/ocserv/ssl/ca.key.pem --load-ca-certificate /etc/ocserv/ssl/ca.cert.pem --load-certificate /etc/ocserv/ssl/revoked/revoked.pem --template /etc/ocserv/ssl/revoked/crl.tmpl --outfile /etc/ocserv/ssl/revoked/crl.pem
```

### 取消吊销

#### 当前仅一个已吊销证书,将其恢复使用

```shell
vi /etc/ocserv/ssl/revoked/revoked.pem
# 删除取消吊销的用户证书,删除后的 revoked.pem 为空文件.
certtool --generate-crl --load-ca-privkey /etc/ocserv/ssl/ca.key.pem --load-ca-certificate /etc/ocserv/ssl/ca.cert.pem --template /etc/ocserv/ssl/revoked/crl.tmpl --outfile /etc/ocserv/ssl/revoked/crl.pem
```

#### 当前有多个已吊销证书,恢复使用其中一个

```shell
vi /etc/ocserv/ssl/revoked/revoked.pem
# 删除取消吊销的用户证书
certtool --generate-crl --load-ca-privkey /etc/ocserv/ssl/ca.key.pem --load-ca-certificate /etc/ocserv/ssl/ca.cert.pem --load-certificate /etc/ocserv/ssl/revoked/revoked.pem --template /etc/ocserv/ssl/revoked/crl.tmpl --outfile /etc/ocserv/ssl/revoked/crl.pem
```

## 编辑模版

### 证书及账号密码登陆参数

注释即禁用,`至少启用一项`.如同时启用则需`双重验证`,其用户名必须和证书中用户名保持一致.

```shell
auth = "certificate"
# 证书登陆
auth = "|AUTH|"
# 账号密码登陆
```

### 证书与账号密码共存

同时启用,并将`auth = "|AUTH|"`修改为`enable-auth = "|AUTH|"`.

```shell
auth = "certificate"
# 证书登陆
enable-auth = "|AUTH|"
# 账号密码登陆
```

### 服务器域名证书

上传域名证书并修改证书路径

```shell
server-cert = /etc/ocserv/server-cert.pem
server-key = /etc/ocserv/server-key.pem
# 上传证书文件并修改证书路径
```

### CA 根证书路径

使用证书登陆时,取消注释并修改证书路径.

```shell
#ca-cert = /etc/ocserv/ca.pem
取消注释启用,修改 CA 根证书路径.
ca-cert = /etc/ocserv/ssl/ca.cert.pem
```

### 证书登陆用户信息

取消注释启用并修改后,后台可查询证书登陆的用户信息

```shell
#cert-user-oid = 0.9.2342.19200300.100.1.1
# 取消注释启用并修改值
cert-user-oid = 2.5.4.3
```

### 吊销列表文件

当生成吊销列表文件时,需启用并修改路径.如没有吊销列表文件,需保持注释禁用.否则会导致服务无法启动.

```shell
#crl = /etc/ocserv/crl.pem
# 取消注释启用,修改 crl.pem 路径.
crl = /etc/ocserv/ssl/revoked/crl.pem
```

## 防火墙

OpenWrt 防火墙需添加以下记录

```shell
iptables -t nat -I POSTROUTING -s 10.10.10.0/24 -j MASQUERADE
iptables -I FORWARD -i vpns+ -s 10.10.10.0/24 -j ACCEPT
iptables -I INPUT -i vpns+ -s 10.10.10.0/24 -j ACCEPT
```

