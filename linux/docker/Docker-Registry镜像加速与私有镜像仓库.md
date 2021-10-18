# Docker-Registry镜像加速与私有镜像仓库

## 简介

> 部署参考链接
> Docker Registry https://docs.docker.com/registry/
> Compose file https://docs.docker.com/compose/compose-file/
> wade&luffy https://www.cnblogs.com/wade-luffy/p/6590849.html

------

## 加速镜像仓库

### 安装服务端

**建议安装至对大陆网络条件优秀的国外 VPS 服务器.**
命令参数过多,为方便配置参数,建议使用`docker compose`部署.

#### docker-compose.yml

> 由于此服务仅用于代理加速下载镜像,数据目录也无必要挂载,可推荐删除整个`volumes:`参数.

```yml
version: "3"
services:
  mirror:
    image: registry:2
    # container_name: registry-mirror
    volumes:
      # 挂载数据目录 (可选)
      - ./mirror:/var/lib/registry
      # 配置时区 (可选)
      - /etc/localtime:/etc/localtime
    ports:
      - 5000:5000
    environment:
      - REGISTRY_PROXY_REMOTEURL=https://registry-1.docker.io
    networks:
      - registry
    restart: always

networks:
  registry:
```

**注意:**环境变量配置上游镜像仓库为官方仓库,请不要修改.除非你明确了解.

```yml
    environment:
      - REGISTRY_PROXY_REMOTEURL=https://registry-1.docker.io
```

#### 启动

```yml
docker-compose up -d
```

成功启动后加速镜像地址为: **101.102.103.104:5000**
可根据需求使用域名配置反向代理,例如: **[https://registry.yourdomain.com](https://registry.yourdomain.com/)**

注意需开放防火墙 5000 端口

### 配置客户端

以`CentOS`为例,创建或修改`/etc/docker/daemon.json`文件.

```json
{
    "registry-mirrors": [
      "http://101.102.103.104:5000"
    ]
}
```

或

```json
{
    "registry-mirrors": [
      "https://registry.yourdomain.com"
    ]
}
```

配置完成需重载 daemon 并重启 docker

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 使用

客户端正常使用 docker pull 或 run 等命令,所有镜像将由加速镜像仓库提供代理拉取.

------

## 私有镜像仓库

### 安装服务端

**本文以部署至公网,开启账号密码,并配置域名反向代理为例.**
命令参数过多,为方便配置参数,建议使用`docker compose`部署.

#### 创建 htpasswd 账号密码

启动一个一次性容器用于创建账号密码.密码文件路径以`/root/registry/htpasswd`为例,账号密码以`admin`和`12345678`为例.

```shell
docker run --rm --entrypoint \
    htpasswd httpd:2 -Bbn \
    admin 12345678 > /root/registry/htpasswd
```

#### docker-compose.yml

> **volumes** 挂载`htpasswd`密码文件,数据目录,时区文件.配置文件`config.yml`作为高级用户可选挂载.
> **environment** 环境变量开启认证,并开启删除镜像功能.

```yml
version: "3"
services:
  registry:
    image: registry:2
    container_name: registry
    volumes:
      # - ./config.yml:/etc/docker/registry/config.yml
      - ./htpasswd:/auth/htpasswd
      - ./registry:/var/lib/registry
      - /etc/localtime:/etc/localtime
    ports:
      - 5000:5000
    environment:
      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm
      - REGISTRY_STORAGE_DELETE_ENABLED=true
    networks:
      - registry
    restart: always

networks:
  registry:
```

#### 启动

```yml
docker-compose up -d
```

成功启动后私有镜像仓库内网地址为: **192.168.1.5:5000**
可根据需求使用域名配置反向代理,例如: **[https://registry.yourdomain.com](https://registry.yourdomain.com/)**

注意需开放防火墙 5000 端口

### 配置客户端

以`CentOS`为例,创建或修改`/etc/docker/daemon.json`文件.

**注意,如使用域名配置反向代理并开启`HTTPS`,则无需配`daemon.json`文件.**

```json
{
    "insecure-registries": [
      "192.168.1.5:5000"
    ]
}
```

或

```json
{
    "insecure-registries": [
      "101.102.103.104:5000"
    ]
}
```

### 使用

#### 登录

```shell
docker login registry.yourdomain.com
docker login 192.168.1.5:5000
docker login 101.102.103.104:5000
# 使用上文创建的账号密码 admin 12345678 登录
```

#### 登出

```shell
docker logout registry.yourdomain.com
docker logout 192.168.1.5:5000
docker logout 101.102.103.104:5000
```

#### Push

将现有镜像 tag 为私有仓库镜像名

```shell
docker images
# 获取现有镜像的 IMAGE ID
docker tag 102816b1ee7d registry.yourdomain.com/mysql:8.0.13
docker tag 102816b1ee7d 192.168.1.5:5000/mysql:8.0.13
docker tag 102816b1ee7d 101.102.103.104:5000/mysql:8.0.13
```

Push 至私有镜像仓库

```shell
docker push registry.yourdomain.com/mysql:8.0.13
docker push 192.168.1.5:5000/mysql:8.0.13
docker push 101.102.103.104:5000/mysql:8.0.13
```

#### Pull

```shell
docker pull registry.yourdomain.com/mysql:8.0.13
docker pull 192.168.1.5:5000/mysql:8.0.13
docker pull 101.102.103.104:5000/mysql:8.0.13
```

#### 查看镜像仓库清单

```shell
curl -u admin:12345678 -X GET https://registry.yourdomain.com/v2/_catalog
```

#### 查看镜像 tag 清单

```shell
curl -u admin:12345678 -X GET https://registry.yourdomain.com/v2/mysql/tags/list
```

#### 删除镜像

```
docker-compose.yml`环境变量中开启`REGISTRY_STORAGE_DELETE_ENABLED=true
```

##### 获取镜像 digest hash

```shell
curl -u admin:12345678 --header "Accept: application/vnd.docker.distribution.manifest.v2+json" -I -X GET https://registry.yourdomain.com/v2/mysql/manifests/8.0.13
# 获取 digest hash 如下
sha256:45a2a291xxx223123fc03d9be551e362b460exxs56787736919baa
```

##### 删除镜像清单

```shell
curl -u admin:12345678 -I -X DELETE https://registry.yourdomain.com/v2/mysql/manifests/sha256:45a2a291xxx223123fc03d9be551e362b460exxs56787736919baa
```

##### 清理磁盘空间

```shell
docker exec registry bin/registry garbage-collect /etc/docker/registry/config.yml
```

##### 手动删除目录

完成上述操作后还可以删除存储目录中的空目录文件,如不删除依旧可以被上述`查看镜像仓库`的命令查询到结果.
**依照上文示例,挂载存储目录路径如下:**

./registry/docker/registry/v2/repositories

------

## 其他

### 参考链接

#### 部署参考链接

> wade&luffy https://www.cnblogs.com/wade-luffy/p/6590849.html

#### 官方参考文档

> Docker Registry https://docs.docker.com/registry/
> Compose file https://docs.docker.com/compose/compose-file/