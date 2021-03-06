# frp服务端配置

## 下载安装

下载

```shell
wget https://github.com/fatedier/frp/releases/download/v0.37.1/frp_0.37.1_linux_amd64.tar.gz
```

解压

```shell
tar -zxvf frp_0.37.1_linux_amd64.tar.gz
```

进入下载的文件夹

```shell
cd frp_0.37.1_linux_amd64
```

创建frps文件夹

```shell
mkdir -p /home/wwwroot/frps
```

移动frps文件

```shell
mv frps frps.ini /home/wwwroot/frps
```

## 配置frps.ini

编辑frps.ini

```shell
vi /home/wwwroot/frps/frps.ini
```

frps.ini参考如下
**请自行修改服务端口,管理端口,管理员账号密码等参数.**

```json
[common]
bind_addr = 0.0.0.0
bind_port = 7000
bind_udp_port = 7001
kcp_bind_port = 7000
vhost_http_port = 80
vhost_https_port = 443
dashboard_addr = 0.0.0.0
dashboard_port = 7500
dashboard_user = jaymz
dashboard_pwd = admin
log_file = ./frps.log
log_level = info
log_max_days = 3
token = crashdada
allow_ports = 2000-3000,3001,3003,4000-50000
max_pool_count = 50
max_ports_per_client = 0
subdomain_host = frps.com
tcp_mux = true
```

## 配置启动

配置开机启动

```shell
vi /lib/systemd/system/frps.service
```

添加以下代码并保存

```json
[Unit]
Description=frps service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/home/wwwroot/frps/frps -c /home/wwwroot/frps/frps.ini

[Install]
WantedBy=multi-user.target
```

```shell
sudo systemctl daemon-reload
```

启动frps

```shell
sudo systemctl start frps
```

配置开机自动启动frps

```shell
sudo systemctl enable frps
```

## 其他命令

```shell
sudo systemctl start frps     # 启动frps
sudo systemctl enable frps    # 开机自启动
sudo systemctl status frps    # 查看状态
sudo systemctl restart frps   # 重启frps
sudo systemctl stop frps      # 关闭frps
netstat -ntlp                 # 查看frps相关端口信息
```

### 检查服务器端安装情况

输入`http://服务器IP:7500`来查看 frps 服务状态

