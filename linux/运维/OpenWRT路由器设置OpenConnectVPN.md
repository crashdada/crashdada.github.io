# OpenWRT 路由器设置OpenConnect VPN

## 服务端

### 前提需求

- OpenWRT 路由器或其他能够部署 OpenConnect VPN 的设备
- 拥有`公网 IP`并配置端口映射

本文以 **OpenWRT** 路由器内网网段 **192.168.1.0** 为例.

### 基本设置

登录`OpenWRT`路由器,打开`服务` - `OpenConnect VPN`.

#### 勾选 Enable server 启动服务

默认端口为`4443`,默认 VPN 网段为`192.168.100.0`,根据情况自行更改,本次修改为`10.10.10.0`.

DNS server 填写当前内网网段的DNS.

#### Routing table

路由表根据需求有两种设置方法

**全局代理** - 当客户端连接 VPN 后,客户端所有`内外网`访问都将通过 VPN 所在的局域网代理.如下图`删除`所有`Routing table`即可.

**内网代理** - 当客户端连接 VPN 后,客户端所有`内网`访问通过 VPN 所在的局域网代理,而`外网`访问则保持使用客户端当前网络.如下图添加`内网`和`VPN`两个网段即可.

此方法连接 VPN 后可访问内网设备,其他互联网访问请求还是走当前网络.**本次选用此种方案。**

### 编辑模版

默认模版并不影响使用,只是如下图在通过`DDNS域名`连接 VPN 时会提示`服务器或证书不受信任`,可以点击`Connect Anyway`或者`更改设置`继续登录.

**可以参考以下步骤部署 DDNS 域名证书.**

模版中查找以下字段:

```shell
server-cert = /etc/ocserv/server-cert.pem
server-key = /etc/ocserv/server-key.pem
```

将`DDNS域名证书`上传并修改路径.例如:

```shell
server-cert = /etc/uhttpd.crt
server-key = /etc/uhttpd.key
```

### 用户设置

在用户设置中创建用于 VPN 连接的账号和密码.`Active users`会显示当前连接的用户状态.

如希望使用证书模式登录,此处可以无需配置用户.

### 防火墙设置

**OpenWRT - 网络 - 防火墙 - 自定义规则**
添加以下三条记录,允许客户端使用路由器转发流量,重启防火墙生效.

```shell
iptables -t nat -I POSTROUTING -s 10.10.10.0/24 -j MASQUERADE
iptables -I FORWARD -i vpns+ -s 10.10.10.0/24 -j ACCEPT
iptables -I INPUT -i vpns+ -s 10.10.10.0/24 -j ACCEPT
```

### 端口映射

默认端口为`4443`,添加**端口映射**到`路由器IP`完成所有配置.

------

## 客户端

下载客户端,使用 DDNS 域名,端口加上账号密码登录.

客户端推荐使用安全,稳定,高效的 **Cisco AnyConnect**.