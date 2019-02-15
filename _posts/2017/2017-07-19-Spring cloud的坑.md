---
layout: post
title: Spring cloud的坑
date: 2017-07-19
categories: 工作
---
在构建eureka-server出现了莫名其妙的错误
This can also happen if you are @ComponentScanning a springframework package (e.g. if you put a @ComponentScan in the default package by mistake)

** WARNING ** : Your ApplicationContext is unlikely to start due to a @ComponentScan of the default package.

血的经验教学 boot 的启动程序一定不要放到跟目录下

一晚上啊，一晚上...最终加了一层package搞定～