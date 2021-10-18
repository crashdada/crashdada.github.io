# 群晖docker部署AdGuard Home

## 前言

尝试在群晖 NAS 的 **docker** 上部署 **AdGurad Home** 实现家庭网络的`广告过滤`和`防止 DNS 劫持`的需求.



AdGuard Home 是一款全网广告拦截与反跟踪软件。在您将其安装完毕后，它将保护您所有家用设备，同时您不再需要安装任何客户端软件。随着物联网与连接设备的兴起，掌控您自己的整个网络环境变得越来越重要。

## 部署

### 创建挂载目录

为`AdGurad Home`创建挂载目录,用于存放数据文件和配置文件.本文以`/docker/AdGuard/work`和`/docker/AdGuard/config`为例.
[![img](https://static.ioiox.com/usr/uploads/2020/04/751277547.jpg)](https://static.ioiox.com/usr/uploads/2020/04/751277547.jpg)

### 启动容器

#### docker - 注册表

搜索`adguard`并下载官方镜像
[![img](https://static.ioiox.com/usr/uploads/2020/04/1777927674.jpg)](https://static.ioiox.com/usr/uploads/2020/04/1777927674.jpg)

#### docker - 映像

启动容器
[![img](https://static.ioiox.com/usr/uploads/2020/04/4141766435.jpg)](https://static.ioiox.com/usr/uploads/2020/04/4141766435.jpg)

#### 高级设置

`勾选`启用自动重启启动
[![img](https://static.ioiox.com/usr/uploads/2020/04/3253128502.jpg)](https://static.ioiox.com/usr/uploads/2020/04/3253128502.jpg)

#### 高级设置 - 卷

分别添加本地文件夹`work`和`config`,分别挂载如下图:

- `work` : **/opt/adguardhome/work**
- `config` : **/opt/adguardhome/conf**

[![img](https://static.ioiox.com/usr/uploads/2020/04/2783687731.jpg)](https://static.ioiox.com/usr/uploads/2020/04/2783687731.jpg)

#### 高级设置 - 端口

配置本地`3000`端口为安装和管理端口,同时配置`TCP`和`UDP`的 DNS 默认`53`端口.
**此处强调一下**: DNS 的默认端口为`53`,而群晖 NAS 一般不会占用`53`端口,所以直接分配固定端口`53`给 AdGuard Home 使用,而`80`管理端口多数被群晖的`反向代理`,`Web Station`等套件占用,此处保持自动分配.下文安装过程中在把管理端口修改为`3000`即可.

[![img](https://static.ioiox.com/usr/uploads/2020/04/4112714823.jpg)](https://static.ioiox.com/usr/uploads/2020/04/4112714823.jpg)

## 配置

### 初始化安装

访问`http://群晖内网IP:3000`开始初始化安装,下文以`192.168.1.4`为例.
[![img](https://static.ioiox.com/usr/uploads/2020/04/3658679696.jpg)](https://static.ioiox.com/usr/uploads/2020/04/3658679696.jpg)

由于群晖 NAS 的`80`端口被占用,将其修改为`3000`.
DNS 服务器的端口默认为`53`,保持默认.

如果修改为其他端口,在后续使用填写 DNS 服务器时还需要加上端口号.

[![img](https://static.ioiox.com/usr/uploads/2020/04/3848875516.jpg)](https://static.ioiox.com/usr/uploads/2020/04/3848875516.jpg)

创建管理账户密码
[![img](https://static.ioiox.com/usr/uploads/2020/04/4137476048.jpg)](https://static.ioiox.com/usr/uploads/2020/04/4137476048.jpg)

下一步
[![img](https://static.ioiox.com/usr/uploads/2020/04/268177710.jpg)](https://static.ioiox.com/usr/uploads/2020/04/268177710.jpg)

安装完成
[![img](https://static.ioiox.com/usr/uploads/2020/04/3402715019.jpg)](https://static.ioiox.com/usr/uploads/2020/04/3402715019.jpg)

### 常规设置

如下图勾选相关选项,也可以根据自己情况选择.
[![img](https://static.ioiox.com/usr/uploads/2020/04/59733384.jpg)](https://static.ioiox.com/usr/uploads/2020/04/59733384.jpg)

**日志配置:**可在`查询日志`页面随时查询各客户端,域名,类型的历史记录,由于部署在群晖 NAS 上,不担心磁盘空间问题,可以设置为`90`天.
**统计配置:**`仪表盘`页面能够非常直观的展示近期的访问统计信息和数据,建议设置`30`或`90`天.
[![img](https://static.ioiox.com/usr/uploads/2020/04/3025735579.jpg)](https://static.ioiox.com/usr/uploads/2020/04/3025735579.jpg)



已阻止的服务根据情况自行选择,多数情况下保持默认不阻止即可.

### DNS 设置

DNS 设置中基本只需要配置`上游 DNS 服务器`和`Bootstrap DNS 服务器`,其他选项在不懂的情况下建议保持默认.

#### 上游 DNS 服务器

部署 **AdGuard Home** 最重要的目的就是为了避免`DNS 劫持`以达到使用`最纯净的 DNS 解析`的目的,其次才是过滤广告的需求.此处的 DNS 服务器列表,建议是同时添加多个国内和国外权威的纯净 DNS 服务器,例如 **Google** , **Cloudflare** , **114dns** 等知名服务商.保证准确性和解析速度.并`勾选`**同时查询功能**.以下为博主使用的服务器列表,以供参考:

```
114.114.114.114
117.50.10.10
119.29.29.29
223.5.5.5
tcp://8.8.8.8
tcp://208.67.222.222
tcp://1.1.1.1
https://cloudflare-dns.com/dns-query
```

[![img](https://static.ioiox.com/usr/uploads/2020/04/3795684128.jpg)](https://static.ioiox.com/usr/uploads/2020/04/3795684128.jpg)

#### Bootstrap DNS 服务器

**Bootstrap DNS 服务器用于解析您指定为上游的 DoH / DoT 解析器的 IP 地址**,此处填写本地最快的 DNS 解析服务器即可,博主使用的是本地电信的 DNS 服务器,大家根据实际情况自行填写,也可以直接使用`114.114.114.114`.设置完应用并测试一下 DNS 是否工作正常.
[![img](https://static.ioiox.com/usr/uploads/2020/04/1478118795.jpg)](https://static.ioiox.com/usr/uploads/2020/04/1478118795.jpg)

#### 访问设置

访问设置一般用户管理和配置规则,默认情况保持为空.如果希望强制不解析部分域名可以在`拦截的域`中添加,请谨慎设置.

### 其他设置

由于此服务仅用于家庭内网,同时有路由器作为 DHCP 服务器,所以`加密设置`,`客户端设置`,`DHCP 设置`我们保持默认无需设置.

## 过滤器

### DNS 封锁清单

DNS 封锁清单即是 AdGuard Home 用于过滤广告和拦截恶意钓鱼网站的关键设置.默认已经有官方的过滤规则,我们可以手动添加更多的规则来达到更好的效果.但也不用添加过多,可能会带来反效果.个人建议使用3-5个常规规则即可,网上已经有很多大神分享和维护的规则.大家可以自行搜索.这里推荐一些常见好用的规则:

**规则太长,点击展开阅读.**



[![img](https://static.ioiox.com/usr/uploads/2020/04/3452153630.jpg)](https://static.ioiox.com/usr/uploads/2020/04/3452153630.jpg)

### 其他规则

多数情况下保持默认即可.
**DNS 允许清单:**配置允许解析的列表,及时这些域名在封锁清单,也会被解析.
**DNS 重写:**用于自行配置 DNS 解析,例如无法 NAT 回流的用户,可以将 DDNS 域名手动解析到内网IP.
**自定义过滤规则:**手动添加过滤规则.

## 使用

### 通过路由器 DHCP 下发配置

常规情况下,家里的客户端一般都是通过路由器的 DHCP 获取 IP ,网关和 DNS 配置.大部分用户默认情况都是使用运营商提供的 DNS ,也有少数用户会手动修改了 DNS 服务器.为了使用 AdGuard Home 来作为 DNS 服务器,需要在 DHCP 设置 DNS 地址为群晖的内网 IP 地址.

值得注意的是,一旦配置好 DHCP 和 DNS 后,群晖或者 AdGuard Home 的 docker 容器一旦重启或者无法使用,则会导致内网无法解析任何域名.如果不是很稳定的环境,建议不要这样设置.

#### 示例截图

由于路由器的不同,设置界面也不同,可以`参考`以下两张截图来配置 DHCP ,其中`192.168.1.4`正是部署 AdGuard Home 的群晖内网 IP,在配置一个公共的备用 DNS 服务器.
[![img](https://static.ioiox.com/usr/uploads/2020/04/87753064.jpg)](https://static.ioiox.com/usr/uploads/2020/04/87753064.jpg)
[![img](https://static.ioiox.com/usr/uploads/2020/04/2052946572.jpg)](https://static.ioiox.com/usr/uploads/2020/04/2052946572.jpg)

### DNS 转发

少数路由器支持 DNS 转发功能,保持 DHCP 的默认配置,由路由器本身提供 DNS 服务,在由路由器转发 DNS 到 AdGuard Home 也可以实现广告拦截和防止 DNS 劫持的需求.

### 手动设置

手动配置 DNS 服务器建议配置多个,首要 DNS 服务器填写 AdGuard Home 所在的群晖内网 IP ,同时在添加一个公共 DNS 服务器备用.

### 使用效果

当成功配置好 DNS 服务器,就可以在 AdGuard Home 的仪表盘查看统计数据,也可以通过查询日志查看详细的过滤情况.

下图的截图为`OpenWrt`路由器固件中的`AdGuard Home`所以原生编译会有更好的支持,能够查看各个客户端的详情.而在`docker`中部署,所有通信都会经过容器的宿主机,所以`客户端`仅仅只会显示`172.17.0.1`的记录,但这并不影响核心功能,只是无法很好的精确查询日志了.



[![img](https://static.ioiox.com/usr/uploads/2020/04/4143234029.jpg)](https://static.ioiox.com/usr/uploads/2020/04/4143234029.jpg)

## 结语

**AdGurad Home** 并不是万能的,也不是所有广告都能过滤和屏蔽.但实际的使用效果相比没有过滤时提升明显,同时还能防止 DNS 劫持.另外`Chrome`用户可以在单独安装`Adblock Plus`或者`uBlock`等插件来更好的提升过滤效果.