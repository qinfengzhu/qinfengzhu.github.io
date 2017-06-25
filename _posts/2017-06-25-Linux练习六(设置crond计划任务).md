---
layout: post
title: "Linux练习六(设置Crond计划任务)"
date: 2017-06-25   
tag: Linux练习 
---


### 设置crond的计划任务

1.查看crond的任务是否开启: systemctl status crond.service

2.crond 的文件位置在 /var/spool/cron/{用户名}

3.crondtab -e 进行编辑crond任务


### 练习试题

1.对natasha 用户配置计划任务,要求在本地时间的每天14:23分执行以下命令 /bin/echo "hi redhat"

```shell

crontab -e -u natasha

--然后进行编辑 i
23 14 * * * /bin/echo "hi redhat"
--然后wq保存退出

```