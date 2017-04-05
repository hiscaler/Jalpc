---
layout: post
title:  "ActiveMQ 安装配置"
date:   2017-04-05
desc: "Windows 中安装配置 ActiveMQ"
keywords: "Java,ActiveMQ,MQ"
categories: [Java]
tags: [Java,ActiveMQ]
icon: icon-java
---

##准备工作
- Java 开发环境安装，完毕后检查通过命令检查是否正确。

　![Java version check]({{ site.img_path }}/install-maven-in-windows10/java-version.png)
- [ActiveMQ 下载](http://activemq.apache.org/download.html)


##配置 Maven 环境

将下载的 ActiveMQ 安装包解压到您要安装的目录，比如“C:\java\activeMQ”，打开命令行，并进入安装目录下的 bin 目录，发现有 win32和 win64 两个文件夹，这2个文件夹分别对应 windows 32 位和 windows 64 位操作系统的启动脚本，根据自己实际情况进入相应的目录，双击 activemq.bat 脚本文件，启动 ActiveMQ。完毕后，我们可以看到如下的界面，表示 ActiveMQ 已经启动完毕。

![ActiveMQ start]({{ site.img_path }}/install-activemq/activemq.start.png)

##验证
ActiveMQ 默认启动到 8161 端口，启动完毕后在浏览器地址栏输入：http://localhost:8161/admin 要求输入用户名密码，默认用户名密码为admin、admin，这个用户名密码是在 conf/users.properties 中配置的。输入用户名密码后便可看到如下图的 ActiveMQ 控制台界面了。


![ActiveMQ WEB Admin]({{ site.img_path }}/install-activemq/activemq.web.admin.png)

##注意点
默认情况下，ActiveMQ 消息存储空间预设为 100GB，临时空间为 50GB，如果您的安装所在盘空间不够的话，可以打开 conf/activemq.xml 配置文件，找到 systemUsage 项目根据实际情况进行设置即可。否则在启动时会抛出一个警告信息。