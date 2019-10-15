---
title: JDWP协议介绍
date: 2019-08-26 10:57:34
description: JDWP协议介绍
categories:
  - 语言
  - java
  - jpda
tags:
  - jpda


---

&emsp;&emsp;JDWP 是 Java Debug Wire Protocol 的缩写，它定义了调试器（debugger）和被调试的 Java 虚拟机（target vm）之间的通信协议。

&emsp;&emsp;这里首先要说明一下 debugger 和 target vm。Target vm 中运行着我们希望要调试的程序，它与一般运行的 Java 虚拟机没有什么区别，只是在启动时加载了 ==Agent JDWP ==从而具备了调试功能。而 debugger 就是我们熟知的调试器，它向运行中的 target vm 发送命令来获取 target vm 运行时的状态和控制 Java 程序的执行。Debugger 和 target vm 分别在各自的进程中运行，他们之间的通信协议就是 JDWP。

&emsp;&emsp;JDWP 与其他许多协议不同，它仅仅定义了数据传输的格式，但并没有指定具体的传输方式。这就意味着一个 JDWP 的实现可以不需要做任何修改就正常工作在不同的传输方式上（在 JDWP 传输接口中会做详细介绍）。

&emsp;&emsp;JDWP 是语言无关的。理论上我们可以选用任意语言实现 JDWP。然而我们注意到，在 JDWP 的两端分别是 target vm 和 debugger。Target vm 端，JDWP 模块必须以 Agent library 的形式在 Java 虚拟机启动时加载，并且它必须通过 Java 虚拟机提供的 JVMTI 接口实现各种 debug 的功能，所以必须使用 C/C++ 语言编写。而 debugger 端就没有这样的限制，可以使用任意语言编写，只要遵守 JDWP 规范即可。JDI（Java Debug Interface）就包含了一个 Java 的 JDWP debugger 端的实现（JDI 将在该系列的下一篇文章中介绍），JDK 中调试工具 jdb 也是使用 JDI 完成其调试功能的。

在这里插入图片描述
![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/3JDWP.md-2019-08-06-15-08-14.png)

## 1. 协议分析

&emsp;&emsp;JDWP 大致分为两个阶段：握手和应答。握手是在传输层连接建立完成后，做的第一件事：

Debugger 发送 14 bytes 的字符串“JDWP-Handshake”到 target Java 虚拟机

Target Java 虚拟机回复“JDWP-Handshake”

&emsp;&emsp;JDWP 的握手协议
在这里插入图片描述
![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/3JDWP.md-2019-08-06-15-08-25.png)

&emsp;&emsp;握手完成，debugger 就可以向 target Java 虚拟机发送命令了。JDWP 是通过命令（command）和回复（reply）进行通信的，这与 HTTP 有些相似。JDWP 本身是无状态的，因此对 command 出现的顺序并不受限制。

&emsp;&emsp;JDWP 有两种基本的包（packet）类型：命令包（command packet）和回复包（reply packet）。

&emsp;&emsp;Debugger 和 target Java 虚拟机都有可能发送 command packet。Debugger 通过发送 command packet 获取 target Java 虚拟机的信息以及控制程序的执行。Target Java 虚拟机通过发送 command packet 通知 debugger 某些事件的发生，如到达断点或是产生异常。

&emsp;&emsp;Reply packet 是用来回复 command packet 该命令是否执行成功，如果成功 reply packet 还有可能包含 command packet 请求的数据，比如当前的线程信息或者变量的值。从 target Java 虚拟机发送的事件消息是不需要回复的。

&emsp;&emsp;还有一点需要注意的是，JDWP 是异步的：command packet 的发送方不需要等待接收到 reply packet 就可以继续发送下一个 command packet。

## 2. JDWP 传输接口（Java Debug Wire Protocol Transport Interface）

&emsp;&emsp;前面提到 JDWP 的定义是与传输层独立的，但如何使 JDWP 能够无缝的使用不同的传输实现，而又无需修改 JDWP 本身的代码？ JDWP 传输接口（Java Debug Wire Protocol Transport Interface）为我们解决了这个问题。

&emsp;&emsp;JDWP 传输接口定义了一系列的方法用来定义 JDWP 与传输层实现之间的交互方式。首先传输层的必须以动态链接库的方式实现，并且暴露一系列的标准接口供 JDWP 使用。与 JNI 和 JVMTI 类似，访问传输层也需要一个环境指针（jdwpTransport），通过这个指针可以访问传输层提供的所有方法。

&emsp;&emsp;当 JDWP agent 被 Java 虚拟机加载后，JDWP 会根据参数去加载指定的传输层实现（Sun 的 JDK 在 Windows 提供 socket 和 share memory 两种传输方式，而在 Linux 上只有 socket 方式）。传输层实现的动态链接库实现必须暴露 jdwpTransport_OnLoad 接口，JDWP agent 在加载传输层动态链接库后会调用该接口进行传输层的初始化。

## 3. 连接管理

&emsp;&emsp;连接管理接口主要负责连接的建立和关闭。一个连接为 JDWP 和 debugger 提供了可靠的数据流。Packet 被接收的顺序严格的按照被写入连接的顺序。

&emsp;&emsp;连接的建立是双向的，即 JDWP 可以主动去连接 debugger 或者 JDWP 等待 debugger 的连接。对于主动去连接 debugger，需要调用方法 Attach

&emsp;&emsp;在连接建立后，会立即进行握手操作，确保对方也在使用 JDWP。因此方法参数中分别指定了 attch 和握手的超时时间。

&emsp;&emsp;address 参数因传输层的实现不同而有不同的格式。对于 socket，address 是主机地址；对于 share memory 则是共享内存的名称。

&emsp;&emsp;JDWP 等待 debugger 连接的方式，首先需要调用 StartListening 方法。该方法将使 JDWP 处于监听状态，随后调用 Accept 方法接收连接

## 4. JDWP 的命令实现机制

&emsp;&emsp;下面将通过讲解一个 JDWP 命令的实例来介绍 JDWP 命令的实现机制。JDWP 作为一种协议，它的作用就在于充当了调试器与 Java 虚拟机的沟通桥梁。通俗点讲，调试器在调试过程中需要不断向 Java 虚拟机查询各种信息，那么 JDWP 就规定了查询的具体方式。

&emsp;&emsp;在 Java 6.0 中，JDWP 包含了 18 组命令集合，其中每个命令集合又包含了若干条命令。那么这些命令是如何实现的呢？下面我们先来看一个最简单的 VirtualMachine（命令集合 1）的 Version 命令，以此来剖析其中的实现细节。

&emsp;&emsp;因为 JDWP 在整个 JPDA 框架中处于相对底层的位置（在前两篇本系列文章中有具体说明），我们无法在现实应用中来为大家演示 JDWP 的单个命令的执行过程。在这里我们通过一个针对该命令的 Java 测试用例来说明。

```java
CommandPacket packet = new CommandPacket(
    JDWPCommands.VirtualMachineCommandSet.CommandSetID,
    JDWPCommands.VirtualMachineCommandSet.VersionCommand);

ReplyPacket reply = debuggeeWrapper.vmMirror.performCommand(packet);

String description = reply.getNextValueAsString();
int    jdwpMajor   = reply.getNextValueAsInt();
int    jdwpMinor   = reply.getNextValueAsInt();
String vmVersion   = reply.getNextValueAsString();
String vmName      = reply.getNextValueAsString();

logWriter.println("description\t= " + description);
logWriter.println("jdwpMajor\t= " + jdwpMajor);
logWriter.println("jdwpMinor\t= " + jdwpMinor);
logWriter.println("vmVersion\t= " + vmVersion);
logWriter.println("vmName\t\t= " + vmName);
```

&emsp;&emsp;这里先简单介绍一下这段代码的作用。

&emsp;&emsp;首先，我们会创建一个 VirtualMachine 的 Version 命令的命令包实例 packet。你可能已经注意到，该命令包主要就是配置了两个参数 : CommandSetID 和 VersionComamnd，它们的值均为 1。表明我们想执行的命令是属于命令集合 1 的命令 1，即 VirtualMachine 的 Version 命令。

&emsp;&emsp;然后在 performCommand 方法中我们发送了该命令并收到了 JDWP 的回复包 reply。通过解析 reply，我们得到了该命令的回复信息。

```
description = Java 虚拟机 version 1.6.0 (IBM J9 VM, J2RE 1.6.0 IBM J9 2.4 Windows XP x86-32
jvmwi3260sr5-20090519_35743 (JIT enabled, AOT enabled)
J9VM - 20090519_035743_lHdSMr
JIT  - r9_20090518_2017
GC   - 20090417_AA, 2.4)
jdwpMajor    = 1
jdwpMinor    = 6
vmVersion    = 1.6.0
vmName       = IBM J9 VM
```

## 5. JDWP 的事件处理机制

&emsp;&emsp;前面介绍的 VirtualMachine 的 Version 命令过程非常简单，就是一个查询和信息返回的过程。在实际调试过程中，一个 JDI 的命令往往会有数条这类简单的查询命令参与，而且会涉及到很多更为复杂的命令。要了解更为复杂的 JDWP 命令实现机制，就必须介绍 JDWP 的事件处理机制。

&emsp;&emsp;在 Java 虚拟机中，我们会接触到许多事件，例如 VM 的初始化，类的装载，异常的发生，断点的触发等等。那么这些事件调试器是如何通过 JDWP 来获知的呢？下面，我们通过介绍在调试过程中断点的触发是如何实现的，来为大家揭示其中的实现机制。

&emsp;&emsp;在这里，我们任意调试一段 Java 程序，并在某一行中加入断点。然后，我们执行到该断点，此时所有 Java 线程都处于 suspend 状态。这是很常见的断点触发过程。为了记录在此过程中 JDWP 的行为，我们使用了一个开启了 trace 信息的 JDWP。虽然这并不是一个复杂的操作，但整个 trace 信息也有几千行。

&emsp;&emsp;可见，作为相对底层的 JDWP，其实际处理的命令要比想象的多许多。为了介绍 JDWP 的事件处理机制，我们挑选了其中比较重要的一些 trace 信息来说明：

```
[RequestManager.cpp:601] AddRequest: event=BREAKPOINT[2], req=48, modCount=1, policy=1
[RequestManager.cpp:791] GenerateEvents: event #0: kind=BREAKPOINT, req=48
[RequestManager.cpp:1543] HandleBreakpoint: BREAKPOINT events: count=1, suspendPolicy=1,
                          location=0
[RequestManager.cpp:1575] HandleBreakpoint: post set of 1
[EventDispatcher.cpp:415] PostEventSet -- wait for release on event: thread=4185A5A0,
                          name=(null), eventKind=2

[EventDispatcher.cpp:309] SuspendOnEvent -- send event set: id=3, policy=1
[EventDispatcher.cpp:334] SuspendOnEvent -- wait for thread on event: thread=4185A5A0,
                          name=(null)
[EventDispatcher.cpp:349] SuspendOnEvent -- suspend thread on event: thread=4185A5A0,
                          name=(null)
[EventDispatcher.cpp:360] SuspendOnEvent -- release thread on event: thread=4185A5A0,
                          name=(null)
```

&emsp;&emsp;首先，调试器需要发起一个断点的请求，这是通过 JDWP 的 Set 命令完成的。在 trace 中，我们看到 AddRequest 就是做了这件事。可以清楚的发现，调试器请求的是一个断点信息（event=BREAKPOINT[2]）。

&emsp;&emsp;在 JDWP 的实现中，这一过程表现为：在 Set 命令中会生成一个具体的 request, JDWP 的 RequestManager 会记录这个 request（request 中会包含一些过滤条件，当事件发生时 RequestManager 会过滤掉不符合预先设定条件的事件），并通过 JVMTI 的 SetEventNotificationMode 方法使这个事件触发生效（否则事件发生时 Java 虚拟机不会报告）。

&emsp;&emsp;当断点发生时，Java 虚拟机就会调用 JDWP 中预先定义好的处理该事件的回调函数。在 trace 中，HandleBreakpoint 就是我们在 JDWP 中定义好的处理断点信息的回调函数。它的作用就是要生成一个 JDWP 端所描述的断点事件来告知调试器（Java 虚拟机只是触发了一个 JVMTI 的消息）。

&emsp;&emsp;由于断点的事件在调试器申请时就要求所有 Java 线程在断点触发时被 suspend，那这一步由谁来完成呢？这里要谈到一个细节问题，HandleBreakpoint 作为一个回调函数，其执行线程其实就是断点触发的 Java 线程。

&emsp;&emsp;显然，我们不应该由它来负责 suspend 所有 Java 线程。

&emsp;&emsp;原因很简单，我们还有一步工作要做，就是要把该断点触发信息返回给调试器。如果我们先返回信息，然后 suspend 所有 Java 线程，这就无法保证在调试器收到信息时所有 Java 线程已经被 suspend。

&emsp;&emsp;反之，先 Suspend 了所有 Java 线程，谁来负责发送信息给调试器呢？

&emsp;&emsp;为了解决这个问题，我们通过 JDWP 的 EventDispatcher 线程来帮我们 suspend 线程和发送信息。实现的过程是，我们让触发断点的 Java 线程来 PostEventSet（trace 中可以看到），把生成的 JDWP 事件放到一个队列中，然后就开始等待。由 EventDispatcher 线程来负责从队列中取出 JDWP 事件，并根据事件中的设定，来 suspend 所要求的 Java 线程并发送出该事件。

&emsp;&emsp;在这里，我们在事件触发的 Java 线程和 EventDispatcher 线程之间添加了一个同步机制，当事件发送出去后，事件触发的 Java 线程会把 JDWP 中的该事件删除，到这里，整个 JDWP 事件处理就完成了。

## 6. 小节

&emsp;&emsp;我们在调试 Java 程序的时候，往往需要对虚拟机内部的运行状态进行观察和调试，JDWP Agent 就充当了调试器与 Java 虚拟机的沟通桥梁。它的工作原理简单来说就是对于 JDWP 命令的处理和事件的管理。由于 JDWP 在 JPDA 中处于相对底层的位置，调试器发出一个 JDI 指令，往往要通过很多 JDWP 命令来完成。

## 7. JDI ：Java 调试接口

&emsp;&emsp;JDI（Java Debug Interface）是 JPDA 三层模块中最高层的接口，定义了调试器（Debugger）所需要的一些调试接口。基于这些接口，调试器可以及时地了解目标虚拟机的状态，例如查看目标虚拟机上有哪些类和实例等。另外，调试者还可以控制目标虚拟机的执行，例如挂起和恢复目标虚拟机上的线程，设置断点等。

&emsp;&emsp;目前，大多数的 JDI 实现都是通过 Java 语言编写的。比如，Java 开发者再熟悉不过的 Eclipse IDE，它的调试工具相信大家都使用过。它的两个插件 org.eclipse.jdt.debug.ui 和 org.eclipse.jdt.debug 与其强大的调试功能密切相关，其中 org.eclipse.jdt.debug.ui 是 Eclipse 调试工具界面的实现，而 org.eclipse.jdt.debug 则是 JDI 的一个完整实现。

## 8. JDI 工作方式

&emsp;&emsp;工作方式：

&emsp;&emsp;首先，调试器（Debuuger）通过 Bootstrap 获取唯一的虚拟机管理器。虚拟机管理器将在第一次被调用时初始化可用的链接器。一般地，调试器会默认地采用启动型链接器进行链接。
调试器调用链接器的 launch () 来启动目标程序，并完成调试器与目标虚拟机的链接
当链接完成后，调试器与目标虚拟机便可以进行双向通信了。
调试器将用户的操作转化为调试命令，命令通过链接被发送到前端运行目标程序的虚拟机上；然后，目标虚拟机根据接受的命令做出相应的操作，将调试的结果发回给后端的调试器；最后，调试器可视化数据信息反馈给用户。
从功能上，可以将== JDI 分成三个部分：数据模块，链接模块，以及事件请求与处理模块==。

&emsp;&emsp;数据模块负责调试器和目标虚拟机上的数据建模；

&emsp;&emsp;链接模块建立调试器与目标虚拟机的沟通渠道；

&emsp;&emsp;事件请求与处理模块提供调试器与目标虚拟机交互方式
