---
layout: post
title:  "Windows 10 中安装配置 Maven"
date:   2017-04-05
desc: "Windows 10 中安装配置 Maven"
keywords: "Java Maven"
categories: [Java]
tags: [Java,Maven]
icon: icon-html
---

准备工作
=======
- Java 开发环境安装，完毕后检查通过命令检查是否正确。
　![Java version check](../static/assets/img/blog/install-maven-in-windows10/java-version.png)
- [Maven 下载](http://maven.apache.org/download.cgi)


配置 Maven 环境
==============
将下载的 Maven 安装包解压到您要安装的目录，比如“C:\java”，接着设置环境变量。依次操作：
我的电脑“右键”　=> 属性 => 高级 => 环境变量
找到系统变量设置处，点击“新建”
![Java version check](../static/assets/img/blog/install-maven-in-windows10/m2_home.setting.png)

这样就完成了 M2_HOME 环境变量设置。

接着，我们还需要设置 Path 环境变量，将 Maven 的 bin 目录加入进去，这样我们就可以在任意目录执行 maven 的命令了。
同样，我们在系统变量中找到 Path 变量名，点击“编辑”，在弹出的菜单中点击“新建”，输入 %M2_HOME%\bin，然后点击确定保存。

验证
===
上述操作完毕后，我们进入命令行，输入 mvn -v 命令，查看是否有如下输出，如有，则安装正常。
![Java version check](../static/assets/img/blog/install-maven-in-windows10/mvn-v.command.png)

> 在 Windows 其他系列版本中安装步骤一样，只是设置环境变量的方式稍有区别而已。


