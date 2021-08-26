---
title: chapter8_Mysql的数据目录
author: pounds
toc: true
mathjax: true
date: 2021-08-26 22:44:12
img:
coverImg: coverage.png
top:
cover:
password:
summary:
tags:
    - MySQL
categories:
    - MySQL是怎样运行的
---
## 8.1 数据库和文件系统的关系:

`Mysql`中像`InnoDB`和`MyISAM`这类存储引擎都是将数据存储在磁盘中,操作系统通过文件系统来管理磁盘,换句话说: 数据是存储在文件系统中的.

读数据是从文件系统中将数据读取到内存中,写数据是从内存中将数据写入到文件系统.

## 8.2 Mysql的数据目录:

Mysql服务器在启动的时候会从文件系统中的某个目录下加载一些数据,之后在运行过程中产生的数据也会存储到这个目录下的某些文件中.这个目录就是`数据目录`.

### 8.2.1 数据目录和安装目录的区别:

`安装目录`: 安装目录是mysql程序运行文件存放的地方

`数据目录`: 数据目录是mysql程序在运行过程中产生的一些 `用户数据`,`程序运行状态数据`等数据存储的地方

### 8.2.2 数据目录在哪儿:

`show variables like 'datadir'`

![image-20210826233812143](chapter8-Mysql的数据目录/image-20210826233812143.png)

