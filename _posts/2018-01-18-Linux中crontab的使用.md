---
layout: post
title: "linux中crontab的使用"
date: 2018-01-18
description: "Linux命令，crontab详解，crontab使用"
tag: Linux学习
---



### crontab安装（centOS）

    yum -y install vixie-cron

### crontab语法（计划任务）

    crontab [-u user] file
        crontab [-u user] [ -e | -l | -r ]
                (default operation is replace, per 1003.2)
        -e      (edit user's crontab) 编辑
        -l      (list user's crontab) 显示所有任务
        -r      (delete user's crontab) 删除
        -i      (prompt before deleting user's crontab)
        -s      (selinux context)

### crontab格式

    使用crontab -e 添加要执行的命令。
    添加的命令必须以如下格式：
            * * * * * /command path

    前五个字段可以取整数值，指定何时开始工作，第六个域是字符串，即命令字段，其中包括了crontab调度执行的命令。 各个字段之间用spaces和tabs分割。

    前5个字段分别表示：
           分钟：0-59
           小时：1-23
           日期：1-31
           月份：1-12
           星期：0-6（0表示周日）

    一些特殊符号：
    *： 表示任何时刻
    ,：　表示分割
    -：表示一个段，如第二端里： 1-5，就表示1到5点
    /n : 表示每个n的单位执行一次，如第二段里，*/1, 就表示每隔1个小时执行一次命令。也可以写成1-23/1.

    一些示例：
    00 8,12,16 * * * /data/app/scripts/monitor/df.sh
    30 2 * * * /data/app/scripts/hotbackup/hot_database_backup.sh
    10 8,12,16 * * * /data/app/scripts/monitor/check_ind_unusable.sh
    10 8,12,16 * * * /data/app/scripts/monitor/check_maxfilesize.sh
    10 8,12,16 * * * /data/app/scripts/monitor/check_objectsize.sh

    43 21 * * * 21:43 执行
    15 05 * * * 　　 05:15 执行
    0 17 * * * 17:00 执行
    0 17 * * 1 每周一的 17:00 执行
    0,10 17 * * 0,2,3 每周日,周二,周三的 17:00和 17:10 执行
    0-10 17 1 * * 毎月1日从 17:00到7:10 毎隔1分钟 执行
    0 0 1,15 * 1 毎月1日和 15日和 一日的 0:00 执行
    42 4 1 * * 　 　 毎月1日的 4:42分 执行
    0 21 * * 1-6　　 周一到周六 21:00 执行
    0,10,20,30,40,50 * * * *　每隔10分 执行
    */10 * * * * 　　　　　　 每隔10分 执行
    * 1 * * *　　　　　　　　 从1:0到1:59 每隔1分钟 执行
    0 1 * * *　　　　　　　　 1:00 执行
    0 */1 * * *　　　　　　　 毎时0分 每隔1小时 执行
    0 * * * *　　　　　　　　 毎时0分 每隔1小时 执行
    2 8-20/3 * * *　　　　　　8:02,11:02,14:02,17:02,20:02 执行
    30 5 1,15 * *　　　　　　 1日 和 15日的 5:30 执行

