# 从pt到家庭媒体中心全攻略

## 基础准备

硬件配置：

- 群晖
- ubuntu服务器
- openwrt

资源情况：

- 零至多个 PT 站账号
- 如果没有 PT 站账号，可以选用公开站点，但是资源有限且保种情况很差

先来看一下网上通用的解决方案：

> 在群晖中套件中心搜索 Emby、transmission、Sonarr、Radarr 等下载安装…然后一番配置之后就可以用了。

但实际上，群晖作为一个存储器，cpu 采用的是 intel atom 系列，算是低端到不行的芯片，内存也只有 4G，一般配置也并没有显卡。而 emby 的媒体分析（ffmpeg）是极度消耗 CPU 和显卡计算能力的。对于种子的搜刮，也需要大量的网络请求。

## 核心

所以我的部署方案是利用 Docker，正好有 `linuxserver` 项目组为这些项目提供了合适的容器镜像。再通过容器挂载的方式进行媒体访问

## 服务介绍

在这里只讲一些重要的服务，搭建过程比较简单。

### Radarr

电影搜刮器。是基于 Sonarr 进行改造开发的，但是作者我感觉人还是不错的，至少目前在我看来这个的功能比 Sonarr 强大得多，但是性质不一样，两者依旧不可缺。说是搜刮器，其实就是调用着别人的搜刮接口~查着别人的电影信息~ 调用别的下载工具~ 把下载好的媒体重命名到合适的文件夹然后，调用别人的播放器~ 但就这一个作为中间桥梁的，一直使用其他人的成果的程序，反倒是不可或缺。

它呢，基本上是以列表连接（imdb、imdb、trakt 等列表）或自己手动搜索添加电影，通过 jackett(现在我用 prowlarr)等支持 Torznab 协议转换的工具搜刮各 bt/pt 站的有效种子，将种子发送到下载服务器并按时监控下载状态。当下载完成后，则会自动将下载好的媒体文件按照预设的方式存储到固定目录中去，并发送通知给用户或其他程序。

### Sonarr

剧集搜刮器好听一点吧。包含了电视剧、电视节目的搜刮。算是这个系列的鼻祖了。但是作者维护一直处于佛系+懒散的状态，奈何我又不会 C#，没有办法自己改动一些东西。不过说起来，剧集搜刮确实要比电影搜刮复杂很多。

跟 Radarr 相比，少了一些功能：

1. 不支持中文
2. 国内 pt 站剧集格式可能单季不会按`S01`这种格式命名，这货就不会自动搜刮。作者解释是为了防止出现把第一季和全剧弄混的情况（大哥就只有一季的电视剧第一季和全剧有什么区别），并且死活不改。
3. 搜刮时的名称不支持中文（甚至源语言），只支持英文。他说他认为所有的 pt 站种子命名格式一定要有标准英文格式，因此不会做额外工作（又来了，tvdb 上名字都全的，搜一下又不会死，我的柯南就因为这个一直没办法自动搜索下载，不过扫黑风暴倒是每一集都有下）。

### emby

不用过多介绍，emby 是一款支持多种视频格式，支持自动搜刮、自动搜索字幕、格式整理，支持各端同步播放状态（播放要钱，状态不要）、多用户管理（权限管理）等等功能。与其作为竞争对手的还有 plex 等，但是都用过之后还是觉得这个更顺手一些。

### qbittorrent、transmission 等

下载器，类似于迅雷。但事实上，迅雷是传说中的吸血软件。bt 的原理是通过大家互相上传下载所需要的文件内容，从而实现下载越多速度越快的原理。而迅雷则是自己建立了一个大的分享网络，将从别人那里拿来的据为己有，只在自己的客户端内传播。（当然，迅雷也自己建立了自己的文件库，所以下载速度才那么快。但是它是不分享给其他下载软件的。）而 pt 站所需要的分享精神也正是其保密、不公开注册的原因。因此，pt 站就只可以用标准 bt 协议即 qb、tr 这种下载工具。据说 qb 抢上传比较厉害，而 tr 资源占用小，适合保种。（pt 相关知识在此不予说明）

### Jackett、Prowlarr

传说中的种子搜刮器。由于国内大部分 PT 站都是很久以前的 nexusPHP 架构，而 pt 的本质大同小异，资源命名也有很固定的规范。因此，这种搜刮器就是利用爬虫的原理对 pt 站进行资源搜索，找到所需要的资源种子返回给客户端。

### 其他

其他服务比如 Lidarr 音乐搜索、Readarr 电子书搜索等服务，国内资源较少，我搭建起来只是为了看起来全面一些，并没太大用处，因此也不会详细讲解。

## 开始搭建

### 群晖的设置（下载器、文件目录及权限）

首先，在群晖上建立一个共享文件夹，在我这里叫做 media，里面存储着所有下载和格式化的媒体文件。在文件夹中建立 downloads、video、music 等目录，分别存储下载资源、格式化后的视频、音乐等资源。

在群晖上安装 transmission，设置默认的下载路径为 media，未完成的下载设定在`/volume1/media/downloads/incomplete` （按自己喜欢）。安装好后，输入 群晖 ip:9091 访问群晖，设置下载路径为 `/volume1/media/downloads/complete`。

通过群晖打开/media/downloads 文件夹，将其权限设置为 sc-tranmission:sc-tranmission 完全访问权限，包含子文件夹。这样才能保证下载过程畅通无阻。

## 目录基础设置

通过远程登录到ubuntu 服务器，找一个自己觉得合适的地方建立一个文件夹。

```
mkdir -p docker/media-center/ 
cd docker/media-center `
```

## docker 的安装

Linux 系统按照 https://docs.docker.com/engine/install/ubuntu/ 的文档来安装，注意切换到自己的系统。

安装好后，根据https://docs.docker.com/compose/install/的说明安装 Docker Compose。

## 挂载磁盘

创建挂载到的文件夹

```shell
sudo mkdir /home/docker/media-center/media && sudo chmod -R 777 /home/docker/media-center/media
```

编辑`/etc/fstab`文件

```shell
sudo vim /etc/fstab
```

在`/etc/fstab`文件中添加自动挂载内容

```shell
//192.168.31.77/media /home/docker/media-center/media cifs rw,dir_mode=0777,file_mode=0777,vers=2.0,username=user,password=psw 0 0
```

最后执行挂载指令

```shell
sudo mount -a
```

> 上述命令解释：
>
> 1. 将上一章在群晖中创建的共享文件夹 media 挂载到 media 文件夹中。
> 2. 在fstab文件添加，自动开机挂载。

## 编写 docker-compose 文件

在当前目录新建 docker-compose.yml 文件，内容如下：

```
version: '3'

services:
  sonarr3:
    image: linuxserver/sonarr:latest
    privileged: True
    restart: unless-stopped
    volumes:
      - ./sonarr:/config
      - ./media:/media
    ports:
      - 8989:8989
    environment:
      - PUID=0
      - PGID=0
      - TZ=Asia/Shanghai
      - UMASK=022

  radarr3:
    image: linuxserver/radarr:nightly
    privileged: True
    restart: unless-stopped
    volumes:
      - ./radarr:/config
      - ./media:/media
    ports:
      - 7878:7878
    environment:
      - PUID=0
      - PGID=0
      - TZ=Asia/Shanghai
      - UMASK=022

  emby:
    image: linuxserver/emby
    ports:
      - 8096:8096
    volumes:
      - ./emby:/config
      - ./media:/media
    restart: unless-stopped
    privileged: True
    devices:
      - /dev/dri:/dev/dri

  prowlarr:
    image: ghcr.io/linuxserver/prowlarr:develop
    container_name: prowlarr
    restart: unless-stopped
    environment:
      - PUID=0
      - PGID=0
      - TZ=Asia/Shanghai
    volumes:
      - ./prowlarr:/config
      - ./media:/media
    ports:
      - 9696:9696
```

各个服务从上至下依次为 sonarr 剧集、radarr 电影、emby 媒体播放、prowlarr 搜刮器。其他的没有写在这里，先让服务运行起来。

执行命令，一段时间的镜像下载后，各个镜像应该启动成功了。

```
sudo docker-compose up -d
```

输入`docker-compose ps`可以查看各个服务的运行情况。