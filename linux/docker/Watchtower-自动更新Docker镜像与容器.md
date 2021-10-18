# Watchtower-自动更新Docker镜像与容器

## 简介

**Watchtower** 是一款实现自动化更新 Docker 镜像与容器的实用工具.它监控着所有正在运行的容器以及相关镜像,当检测本地镜像与镜像仓库中的镜像有差异时,会自动拉取最新镜像并使用最初部署时的参数重新启动相应的容器.

> 项目仓库 [containrrr/watchtower](https://github.com/containrrr/watchtower)
> 镜像仓库 [containrrr/watchtower](https://hub.docker.com/r/containrrr/watchtower)
> 官方文档 [Documentation](https://containrrr.github.io/watchtower/)

## 部署

### 快速启动

以容器的方式启动 Watch­tower

```shell
docker run -d \
    --name watchtower \
    --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower
```

### 清理旧镜像

镜像在更新后旧镜像标签会变为`none`,长期自动更新会导致过多的`none`镜像占用空间,加入`--cleanup`参数可以在每次更新后自动删除`none`镜像.

```shell
docker run -d \
    --name watchtower \
    --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    --cleanup
```

### 指定容器更新

如无需自动更新所有稳定运行的容器,可以配置仅更新指定容器,只需要在命令后加上容器名.例如**只更新**`nginx`和`redis`.

```shell
docker run -d \
    --name watchtower \
    --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    --cleanup \
    nginx redis
```

注意指定容器需填写 **容器名** ,并非镜像名.由于部分容器启动时可能没有定义`--name`参数,请执行`docker ps`查询核对容器名.

### 配置自动更新频率

Watch­tower 默认每 5 分钟轮询一次,可以使用以下参数配置更新的频率.

- `--interval`,`-i` 配置更新周期,默认300秒.
- `--schedule`,`-s` 配置定时更新,使用`Cron表达式`,例如`"0 0 1 * * *"`.即每天凌晨1点更新.

**注意:**当使用`-s`参数来配置定时更新时,由于容器内默认为`UTC`时间,上述设置的`凌晨1点`实际上是北京时间`早上9点`.可以通过加上`-e TZ=Asia/Shanghai`环境变量来定义时区,此时配置的时间则为北京时间.

#### 每小时更新一次

```shell
docker run -d \
    --name watchtower \
    --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    --cleanup \
    -i 3600
```

#### 每天凌晨1点更新（北京时间）

```shell
docker run -d \
    --name watchtower \
    --restart always \
    -e TZ=Asia/Shanghai \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    --cleanup \
    -s "0 0 1 * * *"
```

### 手动更新

使用手动更新的方式,运行一次`Watch­tower`容器来更新所需的容器,更新后会自动删除本次运行的`Watch­tower`容器.只需要加上`--rm`和`--run-once`参数即可.同时也可以配合以上`指定容器`或指`定排除容器`的参数来使用.

#### 手动更新所有容器

```shell
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    --cleanup \
    --run-once
```

#### 手动更新指定容器

```shell
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    --cleanup \
    --run-once \
    nginx redis
```

手动运行更新时会出现以下消息,表示正在更新,请耐心等待几分钟.

```shell
time="2020-02-18T03:58:24Z" level=info msg="Running a one time update."
```

随后提示找到更新镜像,停止容器,更新镜像,重启容器并移除旧镜像.至此更新完毕.

```shell
time="2020-02-18T04:02:45Z" level=info msg="Found new xxxx/xxxx:latest image (sha256:10383f5b5720d7e1fxxxx137034c69b7f6xxxxxxafcc4e9d508b561af77)"
time="2020-02-18T04:02:45Z" level=info msg="Stopping /xxxx (2e9ce1ebe319f3a35d80bxxxxxxxxxx6763ada155da957acb24fe76fc8a8c5) with SIGTERM"
time="2020-02-18T04:02:46Z" level=info msg="Creating /xxxx"
time="2020-02-18T04:02:46Z" level=info msg="Removing image sha256:ff4ee4caaa237174080c0d545xxxxxxxxxxxxxxx5d740ddc51e7737839cb5"
```

## 其他

其他更多参数以及功能请参考官方文档 [Documentation](https://containrrr.github.io/watchtower/)