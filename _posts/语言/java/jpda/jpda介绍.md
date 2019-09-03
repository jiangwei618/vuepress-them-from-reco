---
title: jpda
date: 2019-08-26 10:57:34
description: jpda
categories:
  - 语言
  - java
  - jpda
tags:
  - jpda
export_on_save:
  markdown: true

---

## 1.概述

&emsp;&emsp;Java 程序都是运行在 Java 虚拟机上的，我们要调试 Java 程序，事实上就需要向 Java 虚拟机请求当前运行态的状态，并对虚拟机发出一定的指令，设置一些回调等等，那么 Java 的调试体系，就是虚拟机的一整套用于调试的工具和接口。
对于 Java 虚拟机接口熟悉的人来说，您一定还记得 Java 提供了两个接口体系，JVMPI（Java Virtual Machine Profiler Interface）和 JVMDI（Java Virtual Machine Debug Interface），而它们，以及在 Java SE 5 中准备代替它们的 JVMTI（Java Virtual Machine Tool Interface），都是 Java 平台调试体系（Java Platform Debugger Architecture，JPDA）的重要组成部分。 Java SE 自 1.2.2 版就开始推出 Java 平台调试体系结构（JPDA）工具集，而从 JDK 1.3.x 开始，Java SDK 就提供了对 Java 平台调试体系结构的直接支持。顾名思义，这个体系为开发人员提供了一整套用于调试 Java 程序的 API，是一套用于开发 Java 调试工具的接口和协议。本质上说，它是我们通向虚拟机，考察虚拟机运行态的一个通道，一套工具。理解这一点对于学习 JPDA 非常重要。

&emsp;&emsp;换句话说，通过 JPDA 这套接口，我们就可以开发自己的调试工具。通过这些 JPDA 提供的接口和协议，调试器开发人员就能根据特定开发者的需求，扩展定制 Java 调试应用程序，开发出吸引开发人员使用的调试工具。前面我们提到的 IDE 调试工具都是基于 JPDA 体系开发的，区别仅仅在于它们可能提供了不同的图形界面、具有一些不同的自定义功能。另外，我们要注意的是，JPDA 是一套标准，任何的 JDK 实现都必须完成这个标准，因此，通过 JPDA 开发出来的调试工具先天具有跨平台、不依赖虚拟机实现、JDK 版本无关等移植优点，因此大部分的调试工具都是基于这个体系的。

### 1.1 架构

&emsp;&emsp;JVMTI ( Java VM Tool Interface)：java 1.5 推出的新接口。它通过接口的形式定义了 JVM 可提供的调试服务；

&emsp;&emsp;JDWP(Java Debug Wire Protocol): java 调试通信协议。调试者与被调试者两者通信协议。

&emsp;&emsp;JDI (Java Debug Interface): 高层的 Java 语言接口，调试工具开发者可基于该接口开发自己的调试工具。
在这里插入图片描述

![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/1概述.md-2019-08-06-15-05-23.png)

### 1.2 关于 JPDA 中各层说明

&emsp;&emsp;调试开发者可使用 JPDA 中各层接口进行开发。但是 JDI 是 JPDA 中最高层的，也是使用起来最简单的，鼓励调试工具开发者使用该层接口开发。如果某个公司需要开发调试器，则可参考“JDI 参看实现”。

#### 1.2.1VM

JVM 实现了 JVM TI 接口。
关于 JVM TI 接口的说明见下章节。

#### 1.2.2back-end

&emsp;&emsp;负责和 debugger(调试者）前端通信，并交消息传递给被调试的 VM，并将消息返回给 debugger(调试者）前端。两者通信通过 JDWP（Java Debug Wire Protocol）。back-end 和被调试的虚拟机通信通过 JVMTI （ Java Virtual Machine Debug Interface）通信；
== 前后端的通信机制：==
通信包括两种机制，1）connector，2）transport（具体实现形式没有定义） 。其中 connector 是一个 JDI 对象。backend 与 front-end 可通过 conector 建立通信连接。JPDA 定义了三种 connector:

（1） listening connectors: 前端监听 back-end 的连接请求；
（2）attaching connectors:前端主动的绑定到已经运行了的后端上；
（3）launching connectors:前端自己启动 java vm，其中包括了被调试的 java 代码和 back-end（所有的东西都在一起）

#### 1.2.3JDWP

&emsp;&emsp;JDWP（Java Debug Wire Protocol）是一个为 Java 调试而设计的一个通讯交互协议。
在 JPDA 体系中，作为前端（front-end）的调试者（debugger）进程和后端（back-end）的被调试程序（debuggee）进程之间的交互数据的格式就是由 JDWP 来描述的，它详细完整地定义了请求命令、回应数据和错误代码，保证了前端和后端的 JVMTI 和 JDI 的通信通畅。比如在 Sun 公司提供的实现中，它提供了一个名为== jdwp.dll（jdwp.so）的动态链接库文件==，这个动态库文件实现了一个 Agent，它会负责解析前端发出的请求或者命令，并将其转化为 JVMTI 调用，然后将 JVMTI 函数的返回值封装成 JDWP 数据发还给后端。
另外，这里需要注意的是 JDWP 本身并不包括传输层的实现，传输层需要独立实现，但是 JDWP 包括了和传输层交互的严格的定义，就是说，JDWP 协议虽然不规定我们是通过 EMS 还是快递运送货物的，但是它规定了我们传送的货物的摆放的方式。在 Sun 公司提供的 JDK 中，在传输层上，它提供了 socket 方式，以及在 Windows 上的 shared memory 方式。当然，传输层本身无非就是本机内进程间通信方式和远端通信方式，用户有兴趣也可以按 JDWP 的标准自己实现。

#### 1.2.4JDI

&emsp;&emsp;JDI（Java Debug Interface）是三个模块中最高层的接口，在多数的 JDK 中，它是由 Java 语言实现的。 JDI 由针对前端定义的接口组成，通过它，调试工具开发人员就能通过前端虚拟机上的调试器来远程操控后端虚拟机上被调试程序的运行，JDI 不仅能帮助开发人员格式化 JDWP 数据，而且还能为 JDWP 数据传输提供队列、缓存等优化服务。从理论上说，开发人员只需使用 JDWP 和 JVMTI 即可支持跨平台的远程调试，但是直接编写 JDWP 程序费时费力，而且效率不高。因此基于 Java 的 JDI 层的引入，简化了操作，提高了开发人员开发调试程序的效率。

#### 1.2.5front-end

&emsp;&emsp;front-end 实现了 JDI 接口。
