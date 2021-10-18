# Docker-qiandao自动签到程序安装教程

## 项目介绍

- 签到官网 [https://qiandao.today](https://qiandao.today/)
- 项目地址 https://github.com/binux/qiandao
- Docker https://hub.docker.com/r/fangzhengjin/qiandao/

## Docker 部署

### 挂载路径运行

```shell
cd /root
git clone https://github.com/fangzhengjin/qiandao.git
docker run -d -p 80:80 --name qiandao --restart=always -v /root/qiandao:/usr/src/app fangzhengjin/qiandao
```

### 无挂载路径

```shell
docker run -d -p 80:80 --name qiandao --restart=always daocloud.io/fangzhengjin/qiandao
```

## 配置管理员

使用邮箱注册账号,执行以下命令后 exit 退出容器即可.

```shell
docker exec -it qiandao /bin/bash
# 进入容器
python ./chrole.py 10001@qq.com admin
# 设置管理员
```

## 备份恢复

挂载路径运行仅需备份 `/root/qiandao` 目录即可.

无挂载路径运行需要手动备份和恢复数据库.

### 从容器内拷贝数据库出来

```shell
docker cp qiandao:/usr/src/app/database.db .
```

### 将数据库文件拷贝进容器

```shell
docker cp database.db qiandao:/usr/src/app/
```

### 重启容器

```shell
docker restart qiandao
```

## 备用镜像

国内环境可使用以下镜像

```shell
daocloud.io/fangzhengjin/qiandao
```