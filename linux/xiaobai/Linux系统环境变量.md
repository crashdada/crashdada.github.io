# Linux 系统环境变量

本文将根据作者使用需求,慢慢扩充常用的 Linux 环境变量以便查询使用.

### 目录

```shell
WORK_PATH=$(dirname $(readlink -f $0))
# 仅用于脚本内,代表脚本文件所在目录.
${PWD}
# 执行时所在目录
```

### date

```shell
$(date +%Y)-$(date +%m)-$(date +%d)
# 2021-06-24
$(date +\%Y)-$(date +\%m)-$(date +\%d)
# 如在 crontab 中使用需转译 % 号
$(date "+%Y-%m-%d %H:%M:%S")
# 2021-06-24 13:08:33
```

关于`前一天`的一些写法,参考 [stackoverflow.com](https://stackoverflow.com/questions/15374752/get-yesterdays-date-in-bash-on-linux-dst-safe/15374813) 夏令时问题导致多算了一天的讨论.

```shell
date -d "yesterday 13:00" '+%Y-%m-%d'
# 昨天
# 2021-06-23
date -d "yesterday -1 day" '+%Y-%m-%d'
date -d "yesterday -1 day 13:00" '+%Y-%m-%d'
# 前天
# 2021-06-22
$(date -d "now" +%Y-%m-%d)
# 今天
# 2021-06-24
$(date -d "yesterday" +%Y-%m-%d)
$(date -d "1 day ago" +%Y-%m-%d)
# 昨天
# 2021-06-23
$(date -d "2 day ago" +%Y-%m-%d)
# 前天
# 2021-06-22
$(date -d "3 day ago" +%Y-%m-%d)
# 三天前
# 2021-06-21
$(date -d "15 day ago 2021-06-24" +%Y-%m-%d)
# 具体日期的 15 天前
# 2021-06-09
```