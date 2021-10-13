# ubuntu20.04系统初始化与美化

## 系统初始化
### 更新源

ubuntu20.04默认源连接的是外国服务器，网速特别慢，需要更换为国内服务商提供的源。

```shell
# 备份初始源
sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup
# 编辑sources.list文件
sudo gedit /etc/apt/sources.list
```

将阿里源添加进入源文件

```shell
#  阿里源
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

### 刷新列表

```shell
sudo apt-get update
```

```shell
sudo apt-get upgrade
```

```shell
sudo apt-get install build-essential
```

### 安装输入法

> ubuntu20.04中因为缺少某些包的原因，不能再安装搜狗输入法了。此处使用谷歌输入法。

**简单分为三步：**

- `sudo apt-get install fcitx-googlepinyin`
- 语言支持`language support 中的键盘输入法系统`Keyboard input method system`将默认的iBus，更改为fcitx。
- 应用程序`fcitx配置`中通过加号`+`添加新的输入法`Google Pinyin`。

### 安装常用软件

```shell
sudo apt-get install fcitx-googlepinyin # 谷歌拼音输入法
sudo apt-get install git # git
sudo apt-get install stacer # 系统管理器
sudo apt-get install okular # pdf阅读器
sudo apt-get install pandoc # markdown转word
sudo apt-get install redshift # 屏幕色温调节
sudo apt-get install pdfgrep # pdf文件正则表达式检索
sudo apt-get install recoll poppler-utils # 文件内容检索工具
sudo apt-get install rdfind # 重复文件扫描
```

### 安装chrome

```shell
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

```shell
sudo dpkg -i google-chrome*; sudo apt-get -f install 
```

## 系统美化

[![桌面](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152539526-146055666.png)](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152539526-146055666.png)

### 安装Gnome Tweaks和必需的主题引擎

```shell
sudo apt-get install gtk2-engines-murrine gtk2-engines-pixbuf gnome-tweaks libcanberra-gtk3-module libcanberra-gtk-module libglib2.0-dev gir1.2-gtkclutter-1.0
```

### 下载/安装主题与图标

#### 主题

https://www.pling.com/s/Gnome/p/1241688/

#### 图标

https://www.pling.com/p/1309810/

```shell
# 亮系
sudo cp -r McOS-CTLina-Gnome-1.3.2 /usr/share/themes
```

```shell
# 暗系
sudo cp -r Mc-OS-CTLina-Gnome-Dark-1.3.2 /usr/share/themes
```

```shell
# 透明
sudo cp -r Mc-OS-Transparent-1.3 /usr/share/themes
```

```shell
sudo cp -r Os-Catalina-Night /usr/share/icons
```

```shell
sudo cp -r Os-Catalina-icons /usr/share/icons
```

### 安装主题扩展

```
sudo apt install chrome-gnome-shell
```

#### 火狐安装扩展

```
可以外网的同学可以使用chrome扩展
```

1. 转到[extensions.gnome.org](https://extensions.gnome.org/)，然后单击链接以安装浏览器集成。
   [![安装扩展](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152642295-880590627.png)](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152642295-880590627.png)
2. 单击[此处](https://extensions.gnome.org/extension/19/user-themes/)安装用户主题扩展，以从用户目录加载shell主题。
   [![主题扩展](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152712304-1654882368.png)](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152712304-1654882368.png)
3. （可选）单击[此处](https://extensions.gnome.org/extension/1251/blyr/)以安装扩展程序以应用模糊效果。

### 应用主题和图标

`通过优化工具(Tweak tool)进入`
[![优化](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152739457-1333882923.png)](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152739457-1333882923.png)

### 调整左侧面板

要将左停靠面板移至底部，请运行命令：

gsettings set org.gnome.shell.extensions.dash-to-dock dock-position BOTTOM

要缩短dock面板，请运行：

gsettings set org.gnome.shell.extensions.dash-to-dock extend-height false

将应用启动器图标左移：

gsettings set org.gnome.shell.extensions.dash-to-dock show-apps-at-top true

设置扩展坞面板背景透明度（范围从0到1）：

gsettings set org.gnome.shell.extensions.dash-to-dock background-opacity 0.3

### 隐藏顶部状态栏

首先安装hidetopbar 扩展

```shell
sudo apt-get install gnome-shell-extension-autohidetopbar
```

打开Tweak tool，找到 扩展 --> Hide top bar 扩展，开启即可隐藏顶栏。按windows图标键就会显示出来。
[![隐藏bar](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152811233-1699444086.png)](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152811233-1699444086.png)

[![topbarinfo](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152833067-258934672.png)](https://img2020.cnblogs.com/blog/1054685/202009/1054685-20200901152833067-258934672.png)
卸载，使用命令：`sudo apt-get remove gnome-shell-extension-autohidetopbar`

### 隐藏桌面图标

在优化工具左侧找到扩展选项，然后可以看到在桌面图标右侧有一个设置按钮。点击这个按钮，然后关掉所要隐藏的图标开关。