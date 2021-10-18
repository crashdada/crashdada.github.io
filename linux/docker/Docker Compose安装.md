# Docker Compose安装

1. 官方安装 - 速度慢,有可能被DNS污染导致失败.
2. 手动安装 - 手动下载,离线安装.
3. PIP 在线安装 - 使用 Python 的 pip 包管理工具在线安装.

## 官方安装

官网选择版本 https://github.com/docker/compose/releases
**以下命令手动修改版本号,例如`1.24.1`**

```shell
curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

执行权限

```shell
chmod +x /usr/local/bin/docker-compose
```

检查 docker compose 版本

```shell
docker-compose version
```

## 手动安装

> 官网选择版本 https://github.com/docker/compose/releases
> 选择相应版本,下载到本地或者服务器中.
> 更名为`docker-compose`,并移动到`/usr/local/bin`目录下.

添加执行权限

```shell
chmod +x /usr/local/bin/docker-compose
```

检查 docker compose 版本

```shell
docker-compose version
```

## PIP在线安装

安装依赖

```shell
yum -y install epel-release
```

安装 PIP

```shell
yum -y install python-pip
```

升级 PIP

```shell
pip install --upgrade pip
```

检查 PIP 版本

```shell
pip --version
pip 19.3.1 from /usr/lib/python2.7/site-packages/pip (python 2.7)
```

安装 docker compose ,自行更改版本号.

```shell
pip install -U docker-compose==1.24.1
```

检查 docker compose 版本

```shell
docker-compose version
```

> 如果使用PIP安装时报以下错误,先升级安装requests库.

```
ERROR: Cannot uninstall 'requests'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.
```

升级安装 requests 库

```shell
pip install -I requests==2.10.0
```

再次安装 docker compose

```shell
pip install -U docker-compose==1.24.1
```