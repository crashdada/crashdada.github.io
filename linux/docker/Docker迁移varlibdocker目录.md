# Docker迁移/var/lib/docker目录

## 前言

由于`/var/lib/docker/overlay2`目录占用磁盘空间过大,即使使用`docker system prune -a`命令也无法清理`overlay2`目录下的文件,手动删除则会导致容器无法启动.所以只有将 docker 根目录迁移至空间更大的目录.

## 迁移流程

### 停止 docker 服务

```shell
systemctl stop docker
```

### 迁移目录

创建新的 docker 根目录,确保创建的目录空间容量够大.

```shell
mkdir -p /home/docker/lib
```

迁移`/var/lib/docker`目录至`/home/docker/lib`

```shell
rsync -avz /var/lib/docker /home/docker/lib/
```

配置`devicemapper.conf`,先查看该文件是否存在.如不存在则新建.

```shell
mkdir -p /etc/systemd/system/docker.service.d/
vi /etc/systemd/system/docker.service.d/devicemapper.conf
```

写入以下代码保存:

```conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd  --graph=/home/docker/lib/docker
```

重加载 docker

```shell
systemctl daemon-reload
systemctl restart docker
systemctl enable docker
```

### 检查

执行以下命令检查

```shell
docker info
```

确认显示信息中 Docker Root Dir 路径正确

```json
 Docker Root Dir: /home/docker/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
```

### 完成

再次启动容器,检查无误后可删除原路径`/var/lib/docker/`中的文件.