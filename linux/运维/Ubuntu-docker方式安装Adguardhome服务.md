# Ubuntu docker方式安装AdGuard Home服务

## 安装docker

使用的是阿里云源安装的，具体操作如下：

```shell
#使用 apt-get 进行安装
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce
 
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]
```

安装完成之后, 执行下方命令, 将你的所在用户加入到docker组, 这样运行docker命令就不用每次都使用sudo :

```shell
sudo usermod -aG docker 你的用户名
```

然后重启即可使用docker.

## 安装Portainer

Portainer是一个可视化的docker容器管理平台, 使用Portainer可以很方便地对docker容器进行各种操作, 部署Portainer也很简单, 两条命令即可 :

```shell
docker volume create portainer_data #创建一个portainer_data卷, 由于portainer本身存储的东西不是很重要, 哪怕没了也没关系所以就懒得本地化挂载了.
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

部署完成进入设备的IP : 9000端口即可访问Portainer.

## 安装AdGuard Home

AdGuard Home是一款用于去除广告的开源DNS服务软件, 用户可以在自己的设备上搭建DNS服务用于去除广告和加速访问等等, 在[adguardhome](https://hub.docker.com/r/adguard/adguardhome)的docker hub主页可以看到它的详细信息.

使用下方命令就可以直接拉取AdGuard Home镜像 :

```shell
docker pull adguard/adguardhome
```

需要注意的是, 现在还不能直接使用AdGuard Home提供的容器启动, 因为目前还有2个问题没有解决 :

- DNS服务器需要使用53端口, 但是Ubuntu自身的53端口已经被systemd-resolved占用了, 使用`sudo lsof -i:53`命令可以看到systemd-resolved正在运行中.
- AdGuard Home也需要使用68端口, 但是Ubuntu自身的DHCP会占用68端口, 不过由于我们不使用AdGuard Home的DHCP功能, 所以可以不用管, 换成映射成69端口即可

要解决第一个问题：

```shell
sudo mkdir /etc/systemd/resolved.conf.d #如果没有这个文件夹就创建一个
sudo touch /etc/systemd/resolved.conf.d/adguardhome.conf #创建一个文件
#写入下方内容并且保存
[Resolve]
DNS=127.0.0.1
DNSStubListener=no
#对/etc/resolv.conf进行备份
sudo mv /etc/resolv.conf /etc/resolv.conf.backup
#创建一个链接
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
#重启systemd-resolved
sudo systemctl restart systemd-resolved
```

搞完这套操作, 再执行`sudo lsof -i:53`就看不到占用了.

此时执行下方命令, 即可启动AdGuard Home容器(注意68端口已经被换成69端口) :

```shell
docker run --name adguardhome --restart=always -v /home/jaymz/Docker/adguardhome/work:/opt/adguardhome/work -v /home/jaymz/Docker/adguardhome/conf:/opt/adguardhome/conf -p 53:53/tcp -p 53:53/udp -p 67:67/udp -p 69:68/tcp -p 69:68/udp -p 80:80/tcp -p 443:443/tcp -p 853:853/tcp -p 3000:3000/tcp -d adguard/adguardhome
```

## 初始化AdGuard Home

输入该设备的ip : 3000即可进入配置页面。

### **设置网络接口**

将后台的访问端口更改为 3000，避免与 NAS 后台的 80 端口发生冲突，DNS 端口保持为 53 即可。

### **常规设置**

- 过滤器更新间隔：DNS 过滤清单默认更新间隔，一般为 3 天 - 7 天

- 使用 AdGuard 「浏览安全」网页服务：作用与 Chrome 网页安全性检查类似，开启后，当用户访问存在潜在威胁的网站时，AdGuard 会主动拦截并弹出提示

- 使用 AdGuard 「家长控制」 服务：如果家中有尚未成年的孩子，建议开启，避免访问不良网站

- 强制安全搜索：隐藏 Bing、Google、Yandex、YouTube 网站上 NSFW 等不适宜的内容

- 查询记录保留时间：AdGuard Home 服务端采用 Sqlite 文件数据库存储日志，长时间保留可能会降低运行速度，同时占用大量的储存空间，家庭用户一般保留 24 小时 - 7 天即可

- 统计数据保留时间：用于仪表盘的数据展示，一般保留 24 小时 - 7 天即可

### **DNS 设置**

- 上游 DNS 服务器：AdGuard Home 的上游 DNS 服务器，可参考下方推荐列表，一般保留 1 - 2 个即可。AdGuard Home 除了可以作为广告过滤网关，如果设置了纯净 DNS 后，还可以避免运营商的 DNS 劫持
- BootStrap DNS 服务器地址：作为 DoH / DoT DNS 的前置 DNS 解析器，可参考下方推荐列表
- 查询方式、速度限制、EDNS、DNSSEC、拦截模式、DNS 缓存设置、访问设置可根据需要进行调整，一般保持默认设置即可

### **DNS 服务器推荐**

不同地区连接至 DNS 服务器的速度各有差异，各位可以通过 Ping 测速的方式寻找当地连接延迟最低的 DNS 服务器。更多 DNS 服务器可以在 [AdGuard 文档](https://kb.adguard.com/zh/general/dns-providers)中找到。

| DNS 提供商     | 类别                                 | 地址            |
| -------------- | ------------------------------------ | --------------- |
| 阿里           | IPv4 DNS                             | 223.5.5.5       |
| IPv6 DNS       | 2400:3200:baba::1                    |                 |
| DNS-over-Https | https://dns.alidns.com/dns-query     |                 |
| DNSPod         | IPv4 DNS                             | 119.29.29.29    |
| DNS-over-Https | https://doh.pub/dns-query            |                 |
| 114            | IPv4 DNS                             | 114.114.114.114 |
| Google         | IPv4 DNS                             | 8.8.8.8         |
| IPv6 DNS       | 2001:4860:4860::8888                 |                 |
| DNS-over-Https | https://dns.google/dns-query         |                 |
| Cloudflare     | IPv4 DNS                             | 1.1.1.1         |
| IPv6 DNS       | 2606:4700:4700::1111                 |                 |
| DNS-over-Https | https://dns.cloudflare.com/dns-query |                 |

### **DNS 封锁清单**

为了更好地发挥 AdGuard Home 去广告的功能，仅依靠默认的过滤规则是不够的，但也不宜过多，过多的过滤规则会影响解析的速度，各位可以根据需要添加过滤规则。

| 名称                         | 简介                                                         | 地址                                                         |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| AdGuard DNS Filter           | AdGuard 官方维护的广告规则，涵盖多种过滤规则                 | https://raw.githubusercontent.com/AdguardTeam/FiltersRegistry/master/filters/filter_15_DnsFilter/filter.txt |
| EasyList                     | Adblock Plus 官方维护的广告规则                              | https://easylist-downloads.adblockplus.org/easylist.txt      |
| EasyList China               | 面向中文用户的 EasyList 去广告规则                           | https://easylist-downloads.adblockplus.org/easylistchina.txt |
| EasyPrivacy                  | 反隐私跟踪、挖矿规则                                         | https://easylist-downloads.adblockplus.org/easyprivacy.txt   |
| Halflife 规则                | 涵盖了 EasyList China、EasyList Lite、CJX ’s Annoyance、乘风视频过滤规则，以及补充的其它规则 | https://gitee.com/halflife/list/raw/master/ad.txt            |
| Xinggsf 乘风过滤             | 国内网站广告过滤规则                                         | https://gitee.com/xinggsf/Adblock-Rule/raw/master/rule.txt   |
| Xinggsf 乘风视频过滤         | 视频网站广告过滤规则                                         | https://gitee.com/xinggsf/Adblock-Rule/raw/master/mv.txt     |
| MalwareDomainList            | 恶意软件过滤规则                                             | https://www.malwaredomainlist.com/hostslist/hosts.txt        |
| Adblock Warning Removal List | 去除禁止广告拦截提示规则                                     | https://easylist-downloads.adblockplus.org/antiadblockfilters.txt |
| Anti-AD                      | 命中率高、兼容性强                                           | https://anti-ad.net/easylist.txt                             |
| Fanboy’s Annoyances List     | 去除页面弹窗广告规则                                         | https://easylist-downloads.adblockplus.org/fanboy-annoyance.txt |

以浏览国内网站为主的用户可以使用 anti-AD + Halflife 过滤规则，如有浏览国外网站的需要，可以根据需要添加 AdGuard DNS Filter、Fanboy's Annoyances List 等规则。不同规则之间会存在重叠的情况，可以通过 AdGuard Home 的拦截日志分析哪些规则的使用频率最高，哪些规则拦截频率最低，再加以取舍。

## **部署局域网DNS**

完成 AdGuard Home 的设置后，便可将 AdGuard Home 的 DNS 地址部署到局域网设备上。

### **更改路由器 DNS 地址**

不同品牌路由器修改的方法各有差异，具体步骤可参照说明书或网上的教程（路由器型号 + 更改 DNS）

在局域网设置中找到 DNS 设置，将首选 DNS 服务器更改为 AdGuard Home 的 DNS 地址，可设置为其它的 DNS 服务商，避免因 AdGuard Home 服务器宕机而导致局域网无法访问互联网。更改完成后点击保存即可。在路由器更改 DNS 后，局域网内的所有设备的 DNS 解析都会通过 AdGuard Home DNS 完成，实现过滤广告与反隐私跟踪。

