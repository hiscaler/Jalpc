---
layout: post
title:  "Java 命令"
date:   2017-04-05
desc: "Java 命令简析"
keywords: "java,javac"
categories: [Java]
tags: [Java]
icon: icon-java
---

前言
===
一般情况下，我们都使用 IDE 来编写程序，IDE 为我们提供了自动编译，编写程序后，我们可以直接运行。在自动的背后，IDE 则是运行javac 命令来进行程序的编译。

javac
=====

常见错误：找不到 jar 包

	D:\wwwroot\workspace\activemq-hello-world\src\net\hiscaler>javac HelloQueueConsumer.java
	HelloQueueConsumer.java:3: 错误: 程序包javax.jms不存在
	import javax.jms.Connection;
                ^
	HelloQueueConsumer.java:4: 错误: 程序包javax.jms不存在
	import javax.jms.ConnectionFactory;

这种情况一般是 java 在 classpath 中找不到需要的 jar 包所致，利用命令行编译程序时，手动设置 jar 包路径即可。

	D:\wwwroot\workspace\activemq-hello-world\src\net\hiscaler>javac -cp ".;C:\java\apache-activemq-5.14.4\lib\geronimo-jms_1.1_spec-1.1.1.jar;C:\java\apache-activemq-5.14.4\activemq-all-5.14.4.jar" HelloQueueConsumer.java