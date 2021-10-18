# Docker常规命令

## 常规命令

### 常用

```shell
docker version  
docker info  
docker pull  
docker login  
docker logout  
docker images  
docker ps -a  
docker start|stop|restart  
docker rm xxxx  
docker rmi xxxx  
docker exec -it `name OR id` /bin/bash
```

### 批量停止删除容器和镜像

停止所有容器

```shell
docker stop $(docker ps -aq)
```

删除所有的容器

```shell
docker rm $(docker ps -aq)
```

删除所有的镜像

```shell
docker rmi $(docker images -q)
```

### Docker 目录拷贝

本机`/root/test`目录拷贝到容器内`/root`目录内.

```shell
docker cp /root/test 54c9k13e36eb:/root/
```

本机`/root/test`目录拷贝到容器内,并更名为`/test123`.注意命令后没有`/`.

```shell
docker cp /root/test 54c9k13e36eb:/test123
```

将容器内的`/test`目录拷贝到主机的`/root`目录内.

```shell
docker cp  54c9k13e36eb:/test /root/
```

### Docker 文件拷贝

本机`1.db`文件拷贝到容器内`root`目录内.

```shell
docker cp 1.db 54c9k13e36eb:/root/
```

将容器内的`/root/1.db`文件拷贝到主机的当前目录.

```shell
docker cp 54c9k13e36eb:/root/1.db .
```

## Docker 镜像

### 容器创建镜像

通过容器提交生成镜像

```shell
docker commit 容器名 镜像名
```

保存镜像到文件

```shell
docker save 镜像名 > 文件名.tar
```

从文件恢复到镜像

```shell
docker load < 文件名.tar
```

通过镜像启动新容器

```shell
docker run --name 容器名 相关参数 最后接新的镜像名
```

### 导出导入镜像

导出镜像

```shell
docker images
# 查看 IMAGE ID
docker save 8f544e1c5c82 > xxx.tar
```

导入镜像

```shell
docker load < xxx.tar
```

修改镜像名

```shell
docker tag 8f544e1c5c82 xxx/xxx:latest
```

## Docker Hub

### 上传本地镜像

登录

```shell
docker login
```

上传镜像

```shell
docker push xxxxx/xxxxx:latest
```

### 自动构建镜像

#### Docker Hub 创建仓库

- 链接 GitHub
- 配置 Build Rules

| **Source Type** | **Source**     | **Docker Tag** | **Dockerfile location** |
| --------------- | -------------- | -------------- | ----------------------- |
| Branch          | Master         | Latest         | /                       |
| Tag             | /^v([0-9.]+)$/ | {\1}           | /                       |

#### Tag

##### Github Tag

- `/^v([0-9.]+)$/` 对应 v1.0 或 v1.0.0
- `/^([0-9.]+)$/` 对应 1.0 或 1.0.0
- `/^([0-9]+)$/` 对应 20210101

##### Docker Tag

- `{sourceref}` 对应 Github Tag
- `{\1}` 对应 Github Tag 去除 v

#### GitHub Push

- 正常 commit 并 push 到 GitHub
- Docker Hub 自动构建 master 分支为 latest 镜像
- 本地 tag commit 并 push tag
- Docker Hub 自动构建 tag 标签为 tag名 镜像

##### 创建标签

```shell
git log
# 复制 commit ID
git tag -a v1.0.0 commitID -m "v1.0.0"
# 本地 tag 版本号标签
git push origin v1.0.0 
# 提交本地标签到 GitHub
```

##### 删除标签

```shell
git tag -d 标签名
git push origin :refs/tags/标签名
```

## docker compose

### volumes

顶级卷创建命名卷,并配置命名卷路径.

```yml
volumes:
  database:
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/xxx/xxx'
```