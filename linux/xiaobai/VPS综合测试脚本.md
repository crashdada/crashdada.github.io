# VPS综合测试脚本

这类脚本是综合测试VPS基础信息、硬盘IO、带宽和网络延迟等项目的一键式脚本，主要有：

## 秋水逸冰大佬的Bench.sh脚本

**特点：**

1. 显示当前测试的各种系统信息；
2. 取自世界多处的知名数据中心的测试点，下载测试比较全面；
3. 支持 IPv6 下载测速；
4. IO 测试三次，并显示平均值。

**使用：**

```
wget -qO- bench.sh | bash
#或者
curl -Lso- bench.sh | bash
#或者
wget -qO- 86.re/bench.sh | bash
#或者
curl -so- 86.re/bench.sh | bash
```

Github地址：https://github.com/teddysun/across/blob/master/bench.sh

