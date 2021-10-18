# Docker配置镜像加速及私有仓库

## 配置保存命令

配置`daemon.json`需要重载并重启 Docker 生效

编辑 daemon.json

```shell
vi /etc/docker/daemon.json
```

重新加载 daemon.json

```shell
sudo systemctl daemon-reload
```

重启 Docker

```shell
sudo systemctl restart docker
```

## 配置镜像加速

### 公共镜像加速

添加`Docker 中国官方镜像加速`

```json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

更多公共镜像加速服务

```json
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com",
    "https://registry.docker-cn.com",
    "https://reg-mirror.qiniu.com",
    "https://dockerhub.azk8s.cn",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

### 私有镜像加速

可使用`IP`或`反向代理域名`来配置私有镜像加速服务.
请注意镜像加速的 HTTP/HTTPS 协议,以及开放防火墙端口.

#### 私有域名镜像加速

```json
{
    "registry-mirrors": [
      "https://registry.yourdomain.com"
    ]
}
```

#### 私有 IP 镜像加速

```json
{
    "registry-mirrors": [
      "http://101.102.103.104:5000"
    ]
}
```

## 配置私有仓库

私有仓库可以使用`内网 IP`和`公网 IP`的形式配置.

使用反向代理配置了域名的私有仓库无需配置`daemon.json`.

### 内网 IP

```json
{
    "insecure-registries": [
      "192.168.1.5:5000"
    ]
}
```

### 公网 IP

```json
{
    "insecure-registries": [
      "101.102.103.104:5000"
    ]
}
```

## 同时配置镜像加速和私有仓库

```json
{
    "registry-mirrors": ["https://registry.yourdomain.com"],
    "insecure-registries": ["192.168.1.5:5000"]
}
```

更多的镜像加速服务和私有仓库

```json
{
    "registry-mirrors": [
      "https://registry.yourdomain.com",
      "https://hub-mirror.c.163.com",
      "https://mirror.baidubce.com"
    ],
    "insecure-registries": [
      "192.168.1.5:5000",
      "101.102.103.104:5000"
    ]
}
```

