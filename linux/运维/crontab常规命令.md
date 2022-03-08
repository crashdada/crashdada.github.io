# crontab常规命令

## 基础命令

修改增加

```shell
crontab -e
```

示例 - 每 5 分钟指定目录拉取同步代码

```shell
*/5 * * * * cd /home/wwwroot/www.xxxxx.com/ && git pull
```

列出任务

```shell
crontab -l
```

任务配置写入 cron 下的用户名文件

```shell
cd /var/spool/cron/
ls
cat root
```

## 系统控制

```shell
systemctl start crond.service
systemctl stop crond.service
systemctl restart crond.service
systemctl reload crond.service
systemctl status crond.service
```

## 开机自动启动

编辑 rc.local 文件

```shell
vi /etc/rc.d/rc.local
```

加入以下命令

```shell
systemctl start crond.service
```