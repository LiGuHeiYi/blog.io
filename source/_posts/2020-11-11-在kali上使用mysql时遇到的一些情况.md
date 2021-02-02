---
layout: post
title: 在kali上使用mysql时遇到的一些情况
date: 2020-11-11
categories: blog
tags: [web,kali]
description: 一个有用小知识
typora-copy-images-to: ..\img
---


# 在kali上使用mysql时遇到的一些情况

### 如何重启MySQL，正确启动MySQL

linux平台及windows平台mysql重启方法

#### 　　Linux下重启MySQL的正确方法：

　　1、通过rpm包安装的MySQL

```
　service mysqld restart
```

　　2、从源码包安装的MySQL

　　// linux关闭MySQL的命令

```
$mysql_dir/bin/mysqladmin -uroot -p shutdown
```

　　// linux启动MySQL的命令

```
$mysql_dir/bin/mysqld_safe &
```

　　其中mysql_dir为MySQL的安装目录，mysqladmin和mysqld_safe位于MySQL安装目录的bin目录下，很容易找到的。

　　3、以上方法都无效的时候，可以通过强行命令：“killall mysql”来关闭MySQL，但是不建议用这样的方式，因为这种野蛮的方法会强行终止MySQL数据库服务，有可能导致表损坏

使用这个命令后，输入mysql的结果可能呈现为

ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'

使用下面方法启动就行

　　步骤或方法:RedHat Linux (Fedora Core/Cent OS)

　　1.启动：/etc/init.d/mysqld start

　　2.停止：/etc/init.d/mysqld stop

　　3.重启：/etc/init.d/mysqld restart

　　Debian / Ubuntu Linux

　　1.启动：/etc/init.d/mysql start

　　2.停止：/etc/init.d/mysql stop

　　3.重启：/etc/init.d/mysql restart

#### 　　Windows下重启MySQL的正确方法：

　　1.点击“开始”->“运行”(快捷键Win+R)。

　　2.启动：输入 net start mysql

　　3.停止：输入 net stop mysql



