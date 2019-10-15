---
title: odl 中mxbean 的实现
date: 2019-08-26 10:57:34
description: odl 中mxbean 的实现
categories:
  - 语言
  - java
  - jmx
tags:
  - jmx
  - mxbean
  - odl
---

## 简介

&emsp;&emsp;下图为 odl jmx 监控显示的数据信息。在这里我们能看到 odl 原生暴露给我们的一些信息。主要关注 controller 下面的一下信息

![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/2odl_mxbean.md-2019-08-06-15-00-47.png)

## 实现

org.opendaylight.controller.md.sal.common.util.jmx.AbstractMXBean
