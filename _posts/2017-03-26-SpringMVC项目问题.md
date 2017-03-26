---
layout: post
title: SpringMVC项目问题
date: 2017-03-26
categories: 工作
---

Tomcat 控制台报的错误：

          Caused by: org.springframework.core.NestedIOException:
          ASM ClassReader failed to parse class file - 
          probably due to a new Java class file version that isn’t supported yet:


查找原因：

        项目编译jdk1.8 与 项目 springmvc的版本不匹配。 可通过降低jdk版本到1.7 or lower； 可替换springmvc 版本到4.0以上 