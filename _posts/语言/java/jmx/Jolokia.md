---
title: Jolokia的架构与使用介绍
date: 2019-08-26 10:57:34
description: Jolokia的架构与使用介绍
categories:
  - 语言
  - java
  - jmx
tags:
  - jolokia
  - jmx
---

## 1. jolokia 架构

&emsp;&emsp;虽然 jolokia 是为了满足 JSR-160 的要求，但是他和 JSR-160 连接器有巨大的差异。其中最引人注目的区别是 jolokia 传递数据是无类型的数据（说白了就是使用了 Json 数据传递，替代了 RMI 传递 Java 序列化数据的方式）。

&emsp;&emsp;2003 年提交的 JSR-160 规定客户端可以透明的调用 MBean 服务，无论被调用的 MBean 是驻留在本地还是在远程的 MBean 服务中。这样做的好处是提供了一个简洁通用的 Java API 接口。但是 JSR-160 的实现存在许多问题：

1. 它非常危险，因为它隐性暴露了 JMX 的远程接口。
1. 它还存在性能问题。无论是远程还是本地调用，调用者至少要知道调用过程是怎么样的、会收到什么结果。在实际使用时，需要有明确的远程消息传递模式，让调用者知道现在是在使用响应较慢的远程调用。
1. 使用 RMI（JSR-160 连接器的默认协议栈）时需要使用 Java 对象的序列化与反序列化机制来构建传递管道。这样做就阻碍了 Java 技术栈之外的环境来使用它。
   以上 3 个原因大概就是 RMI（JSR-160 连接器的默认协议栈）在远程传输协议上逐渐失去市场份额的原因。

&emsp;&emsp;Jolokia 是无类型的数据，使用了 Json 这种轻量化的序列化方案来替代 RMI 方案。使用这样的方法当然存在一些缺点（比如需要额外增加一层代理），但是带来了一些优势，至少这样的实现方案在 JMX 世界是独一无二的。

## 2. Jolokia 植入模式（Agent mode）

![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/5Jolokia.md-2019-08-06-15-02-50.png)

&emsp;&emsp;上图展示了 Jolokia 植入模式的体系结构，说明了与之有关的运行环境。

&emsp;&emsp;Jolokia 植入模式是在本地基于 http 协议提供了一个使用 Json 作为数据格式的外部接口，此时 Jolokia 会桥接到本地的 JMX MBeans 接口。Jolokia 使用 http 服务扩展了 JSR-160，因此需要针对 Jolokia 的运行进行一些额外的处理。多种技术可以工作于 http 协议，最常规的方法是将 jolokia 放置到 servlet 容器中，比如 Tomcat 或 Jetty，这样 Jolokia 完全可以看做是一个常规的 Java web 应用，让所有的开发人员都能够很好理解并快速的从中读取数据。

&emsp;&emsp;当然还有更多的方式使用 Jolokia 植入，比如使用 OSGi HttpService 或嵌入到有 Jetty-Server 的应用中。Jvm 代理者需要使用 Java1.6 以上版本，在他运行时，可以连接到任何本地运行的 Java 进程。

&emsp;&emsp;==附注——关于“植入模式”的称呼的说明：官方名为“Agent mode”，按照字面意思应该译为“代理者模式”。但是后面又一个模式叫代理模式（Proxy Mode），为了更便于理解和表达中文意思，这里命名其为“植入模式”。==

## 3. Jolokia 代理模式

&emsp;&emsp;代理模式用于无法将 Jolokia 部署到目标平台上（说白了就是无法部署到同一台服务器）。在这个模式下，唯一可用的方式就是目标服务开启了 JSR-160 连接。这样做大部分是规范原因（原文是“political reasons”——政治原因-\_-）——有时候根本不允许在目标服务器部署一个额外的软件系统，或者是这样做需要等待一个漫长的审批流程。还有一个原因是目标服务器已经通过 RMI 开启了 JSR-160 连接，并且我们不想额外再去在本地部署 Jolokia。

可以将 jolokia.war 部署到 servlet 容器中（这个 war 包也可用于植入模式）。下图是一个典型的代理模式架构。
![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/5Jolokia.md-2019-08-06-15-03-01.png)

一个 jolokia 客户端发送常规的请求到 jolokia 代理服务，这个请求包含了额外的数据用于标记要查询的目标。所有的路由信息包含在请求信息中，使得代理服务无需特别的配置即可工作。

## 4. 结尾

如果没有什么特别的限制，优先使用植入模式。植入模式比代理模式有更多的优势，因为他没有附加层、减少了维度成本和技术复杂性、而且性能也优于代理模式。此外，一些 jolokia 特性也无法在代理模式中使用，例如“merging of MBeanServers”。
