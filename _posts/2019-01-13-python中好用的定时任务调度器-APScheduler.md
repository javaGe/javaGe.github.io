---
layout: post
title: "python中好用的定时任务调度器-apscheduler"
date: 2019-01-13
description: "使用apscheduler可以创建定时任务"
tag: Python学习
---


### 背景

平时想在固定的时间运行某个程序或者python脚本，Windows系统中可以直接使用任务调度器，设置之后到设定时间就会启动，这里有个问题就是，每次都会重新运行。有没有那种直接在代码层面实现任务调度的呢？下面就介绍下**apscheduler**的使用。apscheduler非常的灵活，也能实现类型linux系统中的crontab定时器的功能。

### 调度规则简单应用

**1.date**
最基本的一种调度，作业只会执行一次。它的参数如下：

	run_date (datetime|str) – the date/time to run the job at
	
	timezone (datetime.tzinfo|str) – time zone for run_date if it doesn’t have one already

代码示例：

```python
from datetime import date
from apscheduler.schedulers.blocking import BlockingScheduler
sched = BlockingScheduler()
def my_job(text):
    print(text)
# 只会在2009-11-06 00:00:00 执行一次
sched.add_job(my_job, 'date', run_date=date(2009, 11, 6), args=['text'])
# 只会在2009-11-06 16:30:05执行一次
sched.add_job(my_job, 'date', run_date=datetime(2009, 11, 6, 16, 30, 5), args=['text'])
sched.add_job(my_job, 'date', run_date='2009-11-06 16:30:05', args=['text'])
sched.start()
```

**2.cron**

类似linux中crontab的使用规则

```
year (int|str) – 4-digit year
month (int|str) – month (1-12)
day (int|str) – day of the (1-31)
week (int|str) – ISO week (1-53)
day_of_week (int|str) – number or name of weekday (0-6 or mon,tue,wed,thu,fri,sat,sun)
hour (int|str) – hour (0-23)
minute (int|str) – minute (0-59)
second (int|str) – second (0-59)
start_date (datetime|str) – earliest possible date/time to trigger on (inclusive)
end_date (datetime|str) – latest possible date/time to trigger on (inclusive)
timezone (datetime.tzinfo|str) – time zone to use for the date/time calculations (defaults to scheduler timezone)

```

代码示例：

```python
from apscheduler.schedulers.blocking import BlockingScheduler

def job_function():
    print("Hello World")
# BlockingScheduler
sched = BlockingScheduler()

# 6到8月， 11-12月的第三个星期五的00:00, 01:00, 02:00 和 03:00 启动。
sched.add_job(job_function, 'cron', month='6-8,11-12', day='3rd fri', hour='0-3')

# 周一到周五早上5:30运行，直到2020-05-30截止
sched.add_job(job_function, 'cron', day_of_week='mon-fri', hour=5, minute=30, end_date='2020-05-30')
# 周一到周五，早上6:30运行
sched.add_job(job_function, 'cron', day_of_week='1-5', hour=6, minute=30)
sched.start()
```

**3.interval**
每间隔一段时间运行

```
weeks (int) – number of weeks to wait
days (int) – number of days to wait
hours (int) – number of hours to wait
minutes (int) – number of minutes to wait
seconds (int) – number of seconds to wait
start_date (datetime|str) – starting point for the interval calculation
end_date (datetime|str) – latest possible date/time to trigger on
timezone (datetime.tzinfo|str) – time zone to use for the date/time calculations

```

代码示例：

```python
from datetime import datetime
from apscheduler.schedulers.blocking import BlockingScheduler

def job_function():
    print("Hello World")
# BlockingScheduler
sched = BlockingScheduler()
# 每间隔2小时执行一次
sched.add_job(job_function, 'interval', hours=2)
# 每间隔2小时执行一次， 从2010-10-10 09:30:00开始到2014-06-15 11:00:00截止。
sched.add_job(job_function, 'interval', hours=2, start_date='2010-10-10 09:30:00', end_date='2014-06-15 11:00:00')
# 每间隔一分钟调度一次
scheduler.add_job(job_function, 'interval', minutes=1)
# 每间隔一秒调度一次
scheduler.add_job(job_function, 'interval', seconds=1)
sched.start()
```
