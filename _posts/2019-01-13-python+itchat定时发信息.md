---
layout: post
title: "python+itchat定时发信息"
date: 2019-01-13
description: "使用itchat进行定时发送想要的信息到微信"
tag: Python学习
---

### 背景

在工作中每天下班前都需要在公司的报工系统中进行工作日志的记录。但是有很多同事常常都会忘记报工 。  这种现象也让我们组长感到头疼，也每天在群里面催。就上述问题，我就想能不能每天定时的通知大家报工呢？也因为自己正在学习python知识，所以就用python+itchat+selenium+apscheduler来实现微信群的定时通知来每天通知大家报工。

### 准备

由于使用的是linux系统来运行python程序，所以这里先准备好python在linux系统运行的环境。
**1. python安装：**
		
		linux系统自带python2.7，所以不需要另外安装，如果需要使用python3以上的版本，再进行升级。
		由于这次的程序在版本上没有什么差异，所以无需再安装。

**2. 相应的python库安装：**

	使用的python库有：requests, selenuim, wxpy, apscheduler, itchat  
	由于在linux上没有直接安装这些包的工具，所以现在系统上安装pip, 这样使得我们
	安装python库时更方便。下面简单描述安装pip的过程：
	
	yum install -y  epel-release #先安装epel源
	yum install -y python-pip  # 接着安装pip
	pip install --upgrade pip  # 更新pip到最高版本
	
	安装好pip后，以上的python，直接使用：pip insatll  [对应的库名]  进行安装
	如：pip install  requests

**3. phantomjs安装**

由于需要使用phantomjs进行全屏截图，所以要在linux系统进行安装

	下载安装包：
	wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2
	
	解压缩：
	tar -xjvf phantomjs-1.9.7-linux-x86_64.tar.bz2
	
	建立软链接：
	mv /phantomjs-2.1.1-liunx-x86_64.tar.bz2 /phantomjs  重命名
	ln -s /usr/local/phantomjs/bin/phantomjs /usr/bin/

	安装依赖软件：
	yum -y install fontconfig

	验证是否成功
	phantomjs -v
	

### 代码实现


```
#!/usr/bin/env python
# -*- coding: utf-8 -*- 
import sys 
reload(sys) 
sys.setdefaultencoding('utf8') # 设置默认编码格式为'utf-8'
from selenium import webdriver as wb
import time
from wxpy import *
from apscheduler.schedulers.blocking import BlockingScheduler
import requests

# 登录客户端，这里没有直接使用itchat，而是使用了wxpy,原理差不多。
# cache_path=True :缓存扫码登录的信息，短时间登录不需要重新扫码。
# console_qr=2 :在linux系统中显示二维码。
bot = Bot(cache_path=True, console_qr=2)
# itchat.auto_login()
# itchat.run()

# 获取金山词霸每日一句，英文和翻译
# 报工前发一波鸡汤文。
def get_news():
    try:
        url = "http://open.iciba.com/dsapi"
        r = requests.get(url)
        contents = r.json()['content']
        translation = r.json()['translation']
        return contents, translation
    except Exception as e:
        return '采集鸡汤的程序挂啦！今天没有鸡汤，但还是要报工哟.^-^.'

def screenshots():
    '''
    获取jira上团队工作日志信息，保存为图片，获取失败返回失败信息
    :return:
    '''

    try:
        url = 'http://xxx.xxx.xxx.xxx:9990/secure/deskDomainAction!mainpage.jspa'
        # drive = wb.Chrome()
        driver = wb.PhantomJS()
        driver.implicitly_wait(30)
        driver.get(url)
        print '进入登录页面'

        user_name = driver.find_element_by_id('login-form-username')
        user_name.clear()
        user_name.send_keys('xxx')

        pwd = driver.find_element_by_id('login-form-password')
        pwd.clear()
        pwd.send_keys('xxxx')

        submit = driver.find_element_by_id('login-form-submit')
        submit.click()
        print '登录成功'
        
        # 获取当前句柄
        current_handle = driver.current_window_handle
        # print(current_handle)

        # 登录后，进入工作日志界面
        driver.find_element_by_link_text('工作日志').click()

        # 获取所有的句柄
        handles = driver.window_handles
        # print(handles)

        # 进入第二个句柄,由于界面重新打开了另一个窗口，截图内容在新的窗口
        driver.switch_to_window(handles[1])

        # 保存截图
        driver.get_screenshot_as_file('./test.png')

        print '截图成功！'
        time.sleep(2)
        driver.quit()
    except Exception as e:
        print u'获取工作日志图片失败！'
        driver.quit()
        return '截图程序挂啦！今天没图看了，但还是要准时报工哟！！'

def sendMsg(data):
'''
发送群消息
'''
    # 获取需要发送信息的群
    group = bot.groups().search(u'test')[0]

    group.send(u'@报工一刻@') # 发送固定信息
    group.send(data)  # 发送鸡汤
    # 如果截图失败，发送固定的文字内容。
    try:
        group.send_image('./test.png')  # 发送图片
    except:
        group.send(u'截图程序挂啦！今天没图看了，但还是要准时报工哟！！')
    group.send(u'没报工的赶紧了喂!0..0!')

def job():
    '''
    定时器执行的函数
    :return:
    '''

    # 获取每日news
    news = get_news()

    # # 获取截图
    # msg = screenshots()

    # 发送信息
    sendMsg(news[0]+'\n'+news[1])
    
# 创建调度器，周一到周五，每天17:50 发送报工信息
scheduler = BlockingScheduler()
# scheduler.add_job(job, 'cron', day_of_week='1-5', hour=17, minute=50)
scheduler.add_job(job, 'interval', seconds=30)
scheduler.start()

bot.join() #保证上述代码持续运行
```

### 踩的坑

1.截图后，图片中的中文乱码问题。
解决方法：

	安装相应的字体
	在centos中执行：yum install bitmap-fonts bitmap-fonts-cjk
	在ubuntu中执行：sudo apt-get install xfonts-wqy

2.二维码在终端显示过大问题。

	由于笔记本屏幕过小，登录时，显示的二维码过大，需要下拉，这时需要调整程序中
	Bot类的参数：console_qr = 1,像素单元格的大小。


人生苦短，我用python，学以致用，让我们生活更美好！！！


	