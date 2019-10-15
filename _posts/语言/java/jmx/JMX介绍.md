---
title: Jmx介绍
date: 2019-08-26 10:57:34
description: Jmx介绍
categories:
  - 语言
  - java
  - jmx
tags:
  - jmx
---

## 1.简介

&emsp;&emsp;全称 Java Management Extensions，从 Java5.0 开始引入到标准 Java 技术平台中。JMX 提供了一个标准的方法去管理资源，因为 JMX 是一种动态技术，你可以在被管理资源创建、实例化和实现的时候监控和管理他们。你也可以使用 JMX 技术去监听和管理 Java 虚拟机。

## 2.Why JMX

&emsp;&emsp;JMX 技术使 Java 应用程序无需在管理方面投入大量精力JMX 几乎可以在任何支持 Java 的设备上运行，而且只需要嵌入一个管理对象服务器（MBean Server），使它的一些功能作为一个或几个被管理的 bean(MBean)在目标服务器上注册。然后就可以从管理基础设施中受益。

&emsp;&emsp;JMX 技术提供了一个标准的方法去管理 Java 应用、系统和网络
标准 Java 平台都支持 JMX 架构，因此只要符合 JMX 规范都可以通过 JMX 管理。

&emsp;&emsp;JMX 让为 Java 应用提供了可伸缩、动态、分布式的管理架构
使 Java 应用可以远程管理。

## 3.JMX 架构

![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/1JMX.md-2019-08-06-14-59-25.png)

---

![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/1JMX.md-2019-08-06-15-00-01.png)

## 4.基本术语

&emsp;&emsp;Instrumentation
要管理的资源。使用 Java Bean 描述要管理的资源。这些 Java Bean 叫 MBean（Management Bean）。

&emsp;&emsp;MBean Agent
代理层。主要定义了各种服务以及通信模型，和需要被管理的资源在同一机器上，核心模块是 MBean Server，所有的 MBean 都要向它注册，才能被管理。注册在 MBeanServer 上的 MBean 并不直接和远程应用程序进行通信，他们通过协议适配器（Adapter）和连接器（Connector）进行通信。

&emsp;&emsp;Distributed Layer
也叫 Remote Management Layer. 即远程管理层。MBean Server 依赖于该层的协议适配器（Adaptor）和连接器（Connector），让 JMX Agent 可以被该 JVM 外面的管理系统远程访问。支持多种协议：SNMP，HTML，RMI.

## 5.MBean 在 JDK 中的应用

&emsp;&emsp;代码逻辑主要在 java.lang.management 包中
![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/1JMX.md-2019-08-06-15-00-09.png)

| 平台资源                   | 对应的 MXBean                       | 可使用的数量 |
| -------------------------- | ----------------------------------- | ------------ |
| 缓冲池                     | BufferPoolMXBean                    | 1 个或多个   |
| 类装入系统                 | ClassLoadingMXBean                  | 1 个         |
| 编译系统                   | CompilationMXBean                   | 1 个         |
| VM HotSpotDiagnosticMXBean | 垃圾收集系统 GarbageCollectorMXBean | 至少 1       |
| 内存管理器                 | MemoryManagerMXBean                 | 1 个或多个   |
| 内存                       | MemoryMXBean                        | 1 个         |
| 内存资源                   | MemoryPoolMXBean                    | 1 个         |
| 操作系统                   | OperatingSystemMXBean               | 1 个         |
| logging                    | PlatformLoggingMXBean               | 1 个         |
| 运行时系统                 | RuntimeMXBean                       | 1 个         |
| 系统资源压力               | SystemResourcePressureMXBean        |
| 线程                       | ThreadMXBean                        | 1 个         |

JDK 中提供的这些 MBean 可以通过 ManagementFactory 获取实例。

## 6.MBean 分类

&emsp;&emsp;Standard MBean
&emsp;&emsp;MXBean
&emsp;&emsp;Dynamic MBean
&emsp;&emsp;Open MBean
&emsp;&emsp;Module MBean

### 6.1 MBean 规范

- A set of readable or writable attributes, or both.
- A set of invokable operations.
- A self-description.
- 管理接口贯穿于 MBean 的整个生命周期，并且是不变的。Mbean 在某些预先定义的事件发生时可以发出通知（Notifications）。

### 6.2 Standard MBean

标准 MBean，需要满足规范：

- 定义管理接口 SomethingMBean，并且有一个叫 Something 的实现类。**标准 MBean 是这二者的组合，而且这两个必须在一个包下。**
  例子：

```java
package com.example;
public interface HelloMBean {

    public void sayHello();
    public int add(int x, int y);

    public String getName();

    public int getCacheSize();
    public void setCacheSize(int size);
}

```

```java
package com.example;
public class Hello implements HelloMBean {
    public void sayHello() {
        System.out.println("hello, world");
    }

    public int add(int x, int y) {
        return x + y;
    }

    public String getName() {
        return this.name;
    }

    public int getCacheSize() {
        return this.cacheSize;
    }

    public synchronized void setCacheSize(int size) {
        this.cacheSize = size;
        System.out.println("Cache size now " + this.cacheSize);
    }

    private final String name = "Reginald";
    private int cacheSize = DEFAULT_CACHE_SIZE;
    private static final int
        DEFAULT_CACHE_SIZE = 200;
}
```

### 6.3 MXBean

&emsp;&emsp;与标准 MBean 类似，MXBean 接口进行自述，以及一个实现类。
不同于标准 MBean，实现类可以叫任意名字。

### 6.4 Dynamic MBean

&emsp;&emsp;动态 MBean 即编码方式实现的 MBean。

&emsp;&emsp;不再通过定义接口来进行自述，而是通过实现 DynamicMBean，定义 metadata 来进行描述。

```java
public interface DynamicMBean {

public Object getAttribute(String attribute) throws AttributeNotFoundException,
    MBeanException, ReflectionException;

public void setAttribute(Attribute attribute) throws AttributeNotFoundException,
    InvalidAttributeValueException, MBeanException, ReflectionException ;

public AttributeList getAttributes(String[] attributes);

public AttributeList setAttributes(AttributeList attributes);

public Object invoke(String actionName, Object params[], String signature[])
    throws MBeanException, ReflectionException ;

public MBeanInfo getMBeanInfo();
}
```

- MBeanInfo 描述管理接口
- Attribute 的 getter 和 setter 提供通用属性设置，定义对外暴露的属性
- invoke 提供通用操作，定义对外暴露的操作
- 注意：
  - Dynamic 的意思是管理接口在运行时才会真正的显现出来，不用通过事先静态的接口定义。而不是指可以动态的调整管理接口。
  - MBean 的描述应该是不会改变的，所以一般要在动态 MBean 的构造函数里来构建 MBeanInfo。

### 6.5 Module MBean

&emsp;&emsp;Module MBean 是动态 MBean 的一种实现，所以它的接口、属性、操作也是通过编程方式来定义。

&emsp;&emsp;JMX 规范规定该类必须实现为 javax.management.modelmbean.RequiredModelMBean，管理者要做的就是实例化该类，并配置该构件的默认行为并注册到 JMX 代理中，即可实现对资源的管理。RequiredModelMBean 是一个没有任何管理接口的动态 MBean，但是它可以把 MBeanInfo 跟一个目标对象组合起来。

&emsp;&emsp;目标对象是具体实现管理行为的类。通过 ModelMBean 接口的 setManagedResource()可以将 ModelMBean 和目标对象进行关联。

```java
public void setManagedResource(Object managedResource, String managedResourceType) ;
```

&emsp;&emsp;managedResourceType 的值可以为 ObjectReference, Handle, IOR, EJBHandle 或 RMIReference，但当前只支持 ObjectReference.

![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/1JMX.md-2019-08-06-15-00-32.png)

&emsp;&emsp;Module MBean 具有以下新的特点：

- 持久性。定义了持久机制，可以利用 Java 的序列化或 JDBC 来存储模型 MBean 的状态。
- 通知和日志功能。能记录每一个发出的通知，并能自动发出属性变化通知。
- 属性值缓存。具有缓存属性值的能力。

&emsp;&emsp;使用 Apache commons-modeler 简化 Module MBean 的开发
commons-modeler 支持 xml 描述管理接口，使用动态 MBean 可以配置式开发
参考：http://blog.csdn.net/s464036801/article/details/9980439

## 6.5 Open MBean

&emsp;&emsp;也是一种动态 MBean，规范还在完善中

## 7.Notifications

&emsp;&emsp;MBean 提供了一套通知机制，其实就是观察者模式
继承 javax.management.NotificationBroadcasterSupport 就可以发出通知
实现 javax.management.NotificationListener 可以接收通知

## 8.Agent

&emsp;&emsp;Agent 作为中间代理层，管理着 MBeans 并且对外暴露管理接口，核心模块 MBean Server
通过工厂类：java.lang.management.ManagementFactory 可以创建 MBean Server，然后在 server 上注册 MBean。
如果是只有本地使用，注册完后就可以通过 JConsole 连接管理了。
如果需要远程管理，则需要配置协议适配器提供远程管理能力。

## 9.Remote Management

&emsp;&emsp;远程接口的暴露
通过 com.sun.jdmk.comm.HtmlAdaptorServer 暴露 HTML 远程管理接口
通过 javax.management.remote.JMXConnectorServer 暴露 RMI 远程管理接口

```java
ObjectName adapterName = new ObjectName("MyMBean:name=htmladapter,port=8082");
HtmlAdaptorServer adapter = new HtmlAdaptorServer();
server.registerMBean(adapter, adapterName);
adapter.start();
LocateRegistry.createRegistry(8888);
// 必须service:jmx:开头
JMXServiceURL url = new JMXServiceURL("service:jmx:rmi:///jndi/rmi://localhost:8888/server");
JMXConnectorServer jcs = JMXConnectorServerFactory.newJMXConnectorServer(url, null, server);
jcs.start();
```

## 10.远程客户端

&emsp;&emsp;HTML 方式的远程客户端就是浏览器

&emsp;&emsp;RMI 方式可以使管理客户端，如 JConsole，也可以是自己编写的 Client
项目中的应用

&emsp;&emsp;Spring JMX
http://blog.csdn.net/yaerfeng/article/details/28232435

&emsp;&emsp;log4j JMX
http://logging.apache.org/log4j/2.x/manual/jmx.html

&emsp;&emsp;Tomcat JMX
http://tomcat.apache.org/tomcat-6.0-doc/monitoring.html
