---
layout: post
title: "Linux用户和文件的权限管理"
date: 2018-01-17
description: "linux用户管理，Linux文件权限管理"
tag: Linux学习
---

### 用户权限管理

#### （1）查看用户

    who am i : 查看当前用户
    其他参数：
        -d：打印死掉的进程
        -q：当前用户数和用户名
        -u：当前用户登录信息

#### （2）创建用户

    sudo adduser xxx用户名

    切换用户：
    su :切换到User用户
    sudo：以特权运行cmd命令，需要用户属于sudo组
    su -l xx用户名：切换到该用户

    Ctrl+d：退出当前用户

#### （3）用户组

**查看当前用户属于哪个用户组**

    groups xxx用户

**添加用户到sudo用户组中**

    先切换到root权限用户中
    添加用户到用户组：
    sudo usermod -G sudo ggf

#### （4）删除用户

    sudo deluser ggf --remove-home

### 文件权限管理

#### （1）文件权限解析

![](https://javage.github.io/images/blog/Linux文件权限1.png)

![](https://javage.github.io/images/blog/Linux文件权限2.png)

#### (2) 变更文件的所有者

    在ggf用户下创建一个文件；
    touch file
    它的所有者就是ggf

    切换会root用户，将所有者变更为root
    sudo chown root file

#### (3) 修改文件的权限

**二进制权限的表示**：

![](https://javage.github.io/images/blog/Linux文件权限3.png)

每个文件又三组权限（拥有者，所属用户组，其他用户），对应一个“rwx”,就是一个“7”，
三组就是“777”（最高权限，可读，可写，可执行）

例如：
修改文件为只有拥有者可操作：

    chmod 700 file(文件名)

**加减赋值权限表示：**

g、o 还有 u 分别表示 group、others 和 user，+ 和 - 分别表示增加和去掉相应的权限

实现同上权限：
    chmod go-rw file(文件名)