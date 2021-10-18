# Docker安装

## Docker 官方脚本安装

下载一键安装脚本

```shell
curl -fsSL https://get.docker.com -o get-docker.sh
```

运行一键安装脚本(阿里云加速镜像)

```shell
sudo sh get-docker.sh --mirror Aliyun
```

## Docker 官方手动安装

升级yum

```shell
yum update
```

安装依赖包

```shell
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

配置仓库(国内用户推荐使用下面阿里云加速镜像)

```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

配置仓库(阿里云加速镜像)

```shell
sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

排序查看仓库内文件

```shell
yum list docker-ce --showduplicates | sort -r
```

安装最新版Docker

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

## Docker 启动命令

启动Docker

```shell
sudo systemctl start docker
```

查看Docker版本

```shell
sudo docker version
```

设置Docker开机自动启动

```shell
sudo systemctl enable docker
```

