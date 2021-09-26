# Rclone 相关命令

## 安装配置

### 安装

```shell
curl https://rclone.org/install.sh | sudo bash
# 官网安装脚本
yum -y install fuse
# 安装挂载依赖 fuse
```

### 配置

```shell
rclone config
# 配置挂载网盘
~/.config/rclone/rclone.conf
# 配置文件所在目录
```

### 挂载

```shell
rclone mount <网盘名称>:<网盘路径> <本地路径> <参数> --daemon
# 挂载命令
fusermount -qzu <本地路径>
# 卸载挂载
```

### 参考命令参数

```shell
rclone copy source:path dest:path
# 从源文件拷贝到目的，不会删除目的的文件，会跳过未变更的文件，基于大小和修改时间或MD5值判断是否变更过。
rclone sync source:path dest:path
# 将目的文件夹同步成和原文件夹完成相同，会删除目的文件夹中的其他文件，可以先用 --dry-run 参数运行，查看哪些文件会被删除和变更。
rclone move source:path dest:path
# 移动文件夹，对于少量文件移动到有大量文件的目的，可以使用 --no-traverse 参数加速。
rclone purge remote:pat
# 删除 path 及其中的所有内容。
rclone mkdir remote:path [flags]
# 如果路径不存在，则创建
rclone rmdir remote:path [flags]
# 删除空的存储桶，如果存储桶非空，可以使用 rclone purge
rclone check source:path dest:path [flags]
# 根据文件大小以及 hash 值对源和目的进行校验(md5和sha1)，--size-only 只对比大小；--download 下载下来对比；--one-way，根据源端的数据列表对比； --checksum ，进行 size, checksum 校验；默认，进行大小和修改时间校验。
rclone ls remote:path [flags]
# 查看对象大小和路径；lsl，包括修改时间；lsd，列出目录；lsjson，以 json 格式输出。
rclone lsd remote:path [flags]
# 列出路径下的目录或存储桶
rclone delete remote:path [flags]
# 删除 path 下符合条件的对象
rclone size remote:path [flags]
# 查看远端的文件数目和总大小
rclone rcat remote:path [flags]
# 将标准输出复制到远程文件中
```

## 挂载实例

### OneDrive

#### 创建 OneDrive API

OneDrive API 的获取如需要图文教程,可参考以下链接,但请注意仅仅只是参考图片找到相对的位置,具体详细的配置仍需仔细参考本文的各项参数.
https://www.ioiox.com/archives/103.html

- 访问 [Azure](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) 注册
- 账户类型选择`任何组织目录中的帐户和个人 Microsoft 帐户`
- 重定向 URI 为`http://localhost`
- 注册跳转登陆后获取应用程序 ID 即`Client_ID`
- 证书和密码中添加新客户端密码即`Client_secret`
- 设置 API 权限 Files.Read、Files.ReadWrite、Files.Read.All、Files.ReadWrite.All、offline_access、User.Read 权限.
- 使用 Windows [下载客户端](https://rclone.org/downloads/) 解压并进入目录,在地址栏输入`CMD`启动命令提示符,使用以下命令获取 API .

```shell
rclone authorize "onedrive" "Client_ID" "Client_secret"
# 获取 Token
# 以下为获取的 Token 示例
{"access_token":"xxxxxxxxxxxxx","expiry":"2020-06-18T11:29:12.482633133+08:00"}
```

#### Rclone 配置连接 OneDrive

```shell
rclone config
# 配置连接
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
# 选择 n 创建
name> OneDrive
# 添加自定义网盘名称
...
23 / Microsoft OneDrive
   \ "onedrive"
...
Storage> 23
# 列出所有支持的网盘列表,选择 OneDrive 所在的 23 号
client_id> xxxxxxxxxxxxxxxxxxxxx
# 粘贴创建 API 时生成的 Client_ID
client_secret> xxxxxxxxxxxxxxxxxxxxx
# 粘贴创建 API 时生成的 Client_secret
Edit advanced config? (y/n)
y) Yes
n) No (default)
y/n> n
# 选 n 回车
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes (default)
n) No
y/n> n
# 选 n 回车
result> {"access_token":"xxxxxxxxxxx","expiry":"2020-06-18T11:29:12.482633133+08:00"}
# 此处粘贴上文利用创建的 API 使用 Windows rclone 生成的完整 Token
Choose a number from below, or type in an existing value
 1 / OneDrive Personal or Business
   \ "onedrive"
 2 / Root Sharepoint site
   \ "sharepoint"
 3 / Type in driveID
   \ "driveid"
 4 / Type in SiteID
   \ "siteid"
 5 / Search a Sharepoint site
   \ "search"
Your choice> 1
# 选择 OneDrive 类型
Found 1 drives, please select the one you want to use:
0: OneDrive (business) id=xxxxxxxxxxxxxx
Chose drive to use:> 0
# 找到 1 个网盘,选择编号 0
Found drive 'root' of type 'business', URL: https://xxx.sharepoint.com/personal/xxx_xxx_onmicrosoft_com/Documents
Is that okay?
y) Yes (default)
n) No
y/n> y
# 选择 y 确认完成
--------------------
[OneDrive]
type = onedrive
client_id = xxxxxxxxxxxxxxxxxxxxx
client_secret = xxxxxxxxxxxxxxxxxxxxx
token = {"access_token":"xxxxxxxxxxx","expiry":"2020-06-18T11:29:12.482633133+08:00"}
drive_id = xxxxxxx
drive_type = business
--------------------
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y
# 列出当前网盘信息,选择 y 最终确认.
Current remotes:

Name                 Type
====                 ====
OneDrive             onedrive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
# 回到配置初始界面,并列出当前网盘列表,选择 q 退出.
```

#### 挂载网盘

```shell
mkdir /data
# 创建本地挂载目录
rclone mount OneDrive: /data --copy-links --allow-other --allow-non-empty --umask 000 --daemon
# 挂载命令
# 其中 OneDrive: 为挂载网盘根目录
# 其中 OneDrive:/xxx 为挂载网盘子目录.
fusermount -qzu /data
# 卸载挂载
```

### 腾讯云 COS / S3

腾讯云 COS 对象存储官方推出的`COSFS`在部分场景下使用体验不佳,由于 COS 支持标准`S3`协议,可以使用 Rclone 来挂载.

#### 配置文件

编辑`~/.config/rclone/rclone.conf`配置文件,添加并修改以下配置参数:

```shell
[cos]
# 自定义挂载名称
type = s3
provider = Other
env_auth = false
# 不从环境变量中获取密钥
access_key_id = AKIDXXXXXXXXXXXXXX
# 腾讯云的 secretId
secret_access_key = YYYYYYYYYYYYYYYYYYY
# 腾讯云的 secretKey
endpoint = cos.ap-shanghai.myqcloud.com
# COS 区域域名
```

#### 挂载

```shell
mkdir /cos
# 创建本地挂载目录
rclone mount cos:test-1251668577 /cos --copy-links --allow-other --allow-non-empty --umask 000 --daemon
# 挂载命令
# 其中 cos:test-1251668577 为挂载名为 cos 的对象存储中的 test-1251668577 存储桶 
# 其中 cos:test-1251668577/xxx 为 挂载名为 cos 的对象存储中的 test-1251668577 存储桶的子目录.
fusermount -qzu /cos
# 卸载挂载
```

#### 参考命令参数

```shell
rclone lsd cos:
# 查看地域存储桶
rclone mkdir cos:rclone-test-1251668577
# 创建存储桶rclone-test-1251668577
rclone sync local-folder/ cos:rclone-test-1251668577/storage
# 将本地local-folder下的文件同步到存储桶的/storage目录下，该操作会将/storage下的所有其他文件删除掉
rclone ls cos:rclone-test-1251668577
# 列出rclone-test-1251668577根目录下的文件
rclone copy local-folder/ cos:rclone-test-1251668577/
# 拷贝本地文件或目录到COS上，不会删除目的端的其他文件
rclone copy cos:rclone-test-1251668577 cos:rclone-test-backup-1251668577
# 同一个存储，在服务端使用copy操作拷贝文件
rclone sync local-folder/ cos:rclone-test-1251668577/ --backup-dir cos:rclone-test-backup-1251668577/20191011
# 将本地文件同步到cos，并备份被删除或修改的文件到备份存储桶中
rclone copy --max-age 24h --progress --no-traverse local-folder/ cos:rclone-test-1251668577/
# --max-age 24h过滤出来最近24小时变更过的文件，--progress显示拷贝进度，--no-traverse在从源拷贝少量文件到目的中大量目的文件时，速度会更快
rclone check local-folder/ cos:rclone-test-1251668577/
# --one-way，查看本地文件是否都同步到了目的端，默认校验修改时间和大小
rclone --min-size 500B lsl  cos:rclone-test-1251668577/
# 查看存储桶中500B以上的文件列表
rclone --dry-run --min-size 300B delete cos:rclone-test-1251668577/
# 查看存储桶中500B以上的待删除文件列表
rclone delete oss:oss-test-bucket-1215715707/ --include=/stl-views.gdb
# 删除根目录下的stl-views.gdb文件，如果不带/前缀，则会删除所有stl-views.gdb文件
rclone size cos:rclone-test-1251668577/
# 查看存储桶中对象数目和占用的空间大小
rclone mount cos:rclone-test-1251668577/ rclone-mnt/
# 将cos挂载成一个本地文件系统
rclone ncdu cos:rclone-test-1251668577/
# 一个简易文本形式的文件浏览器，用于存储桶中的文件浏览、文件和文件夹删除等操作
rclone cat cos:rclone-test-1251668577/test.cpp --head 10
# 输出test.cpp的前10个字节
echo "hello world" |rclone rcat cos:rclone-test-1251668577/rcat.txt
# 将标准输出复制到存储桶的rcat.txt文件中，会覆盖目标文件
rclone sync oss:oss-test-bucket-1215715707/  cos:rclone-test-1251668577/ -P
# 同步oss存储桶中的数据到cos存储桶中，-P选项显示进度
rclone check oss:oss-test-bucket-1215715707/  cos:rclone-test-1251668577/ -P
# 进行数据对比校验
rclone md5sum cos:rclone-test-1251668577/
# 为所有文件生成MD5值
rclone tree cos:rclone-test-1251668577/ -C -D
# 显示文本格式的目录树结构，-C选项带颜色显示，-D显示上次修改时间
```

## 参考链接

- [Rclone 官网文档](https://rclone.org/docs/)
- [使用 Rclone 访问腾讯云 COS 教程](https://cloud.tencent.com/developer/article/1520867)

