---
layout: post
title: "Linux 下定时任务命令 crontab 使用指南"
description: ""
category: Linux基础
tags: [bash, Linux, command]
---

Linux 定时任务命令 `crontab` 很简单实用，用来实现周期性或者定时执行某条命令或者脚本，例如可以实现监控、定时备份数据等，在运维中很有作用。

## crontab 命令格式

``` text
Usage:
 crontab [options]

Options:
 -u <user>  define user (定义任务运行的用户，不加默认为当前登录用户)
 -e         edit user's crontab (用来修改定时任务文件，打开一个临时文件，写入保存之后会进行定时任务安装)
 -l         list user's crontab （列出用户的定时任务）
 -r         delete user's crontab （删除用户的定时任务）
 -i         prompt before deleting （删除之前进行提示）
```

* **列出用户的定时任务**：

``` bash
[root@ecs-guohailn ~]# crontab -l
0,30  1-06 * * * /etc/init.d/smb restart
[root@ecs-guohailn ~]# crontab -l -u guohailin
* * * * * echo 'xxx' > /dev/console
```

* **添加定时任务**有多种方法：

1. 可以使用 `-e` 参数，会进行编辑一个临时文件，按照 crontab 的文件格式进行添加，建议在定时任务前加上注释以方便其他人能看懂该定时任务的用途，退出并保存时会写入到 crontab 的配置文件中，在 `/var/spool/cron/username` 文件中。
2. 当然你也可以直接打开 `/var/spool/cron/username` 文件进行编辑保存，但相比推荐第一种方法，因为会进行 crontab 文件格式的语法校验。
3. 将定时任务写到一个文件中，再使用 `crontab file` 命令来安装定时任务，注意这种方法会将原定时任务给覆盖掉。

## crontab 文件格式

crontab 文件每条非注释行均是一个定时任务，其格式为：

```
<Minute> <Hour> <Day_of_the_Month> <Month_of_the_Year> <Day_of_the_Week> <command>

* * * * *
| | | | |
| | | | +---- Day of the Week   (range: 1-7, 1 standing for Monday)
| | | +------ Month of the Year (range: 1-12)
| | +-------- Day of the Month  (range: 1-31)
| +---------- Hour              (range: 0-23)
+------------ Minute            (range: 0-59)
```

最后一个字段表示命令，可以是系统命令，也可以是用户写的shell脚本。前面五个字段组成了定时任务的时间格式：

* 每个字段根据其含义有其取值范围，如果用 `*` 占位，表示**每**的意思，例如 `* * * * *` 表示每分钟执行
* 还可以是具体的值，或者用 `-` 表示一个值的范围，或者用 `,` 隔开的多个取值，例如 `0-14,30-44 * 1,5 * *` 表示每月1号和5号每小时的第一刻钟和第三刻钟执行
* 还可以用 `/` 取时间的间隔频率，例如 `*/2 [0-12]/2 * * *` 表示每天上午每两小时每两分钟执行


## crontab 调试

定时任务在 Linux 系统是有个服务 `crond` 来运行的，可以使用命令 `service crond status/start/stop/restart/reload` 来启停、查看任务状态等。

同时 `crond` 服务的日志打在 `/var/log/cron` 文件中，调试时可以查看该日志看任务是否定时调用。
