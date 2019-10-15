---
title: jmx中的mxben
date: 2019-08-26 10:57:34
description: jmx中的mxben
categories:
  - 语言
  - java
  - jmx
tags:
  - jmx
  - mxbean
---

## 1. 虚拟机 Java 活动应用服务器 Bean

&emsp;&emsp;JAVA 平台 MXBean 是一种托管 Bean，它符合 JMX Instrumentation Specification,提供管理接口，用于监视和管理 Java 虚拟机以及 Java 虚拟机在其上运行的操作系统。

**JAVA 平台 MXBean:**

| bean                   | 作用                                                    |
| ---------------------- | ------------------------------------------------------- |
| ClassLoadingMXBean     | 用于 Java 虚拟机的类加载系统的管理接口。                |
| CompilationMXBean      | 用于 Java 虚拟机的编译系统的管理接口。                  |
| GarbageCollectorMXBean | 用于 Java 虚拟机的垃圾回收的管理接口。                  |
| MemoryManagerMXBean    | 内存管理器的管理接口。                                  |
| MemoryMXBean Java      | 虚拟机的内存系统的管理接口。                            |
| MemoryPoolMXBean       | 内存池的管理接口。                                      |
| OperatingSystemMXBean  | 用于操作系统的管理接口，Java 虚拟机在此操作系统上运行。 |
| RuntimeMXBean Java     | 虚拟机的运行时系统的管理接口。                          |
| ThreadMXBean Java      | 虚拟机线程系统的管理接口。                              |

&emsp;&emsp;Java 虚拟机的每个平台 MXBean 都具有惟一的 ObjectName，以在平台 MBeanServer 中注册。Java 虚拟机具有以下管理接口的单一实例：
管理接口 | 对象名称
---|---
ClassLoadingMXBean | java.lang:type=ClassLoading
MemoryMXBean | java.lang:type=Memory
ThreadMXBean | java.lang:type=Threading
RuntimeMXBean | java.lang:type=Runtime
OperatingSystemMXBean | java.lang:type=OperatingSystem
MemoryManagerMXBean | java.lang:type=MemoryManager

## 2. 获得平台 MXBean 的方法:

1. 通过调用 ManagementFactory.getClassLoadingMXBean() 方法
2. 平台 MBeanServer 方法获得.

```java
    MBeanServer server = ManagementFactory.createPlatformMBeanServer();
    MBeanInfo info = server.getMBeanInfo(new ObjectName("java.lang:type=ClassLoading")); //
    通过ClassLoading的ObjectName,查找MBean为管理而公开的属性和操作。
    server.getAttribute(new ObjectName("java.lang:type=ClassLoading"), "TotalLoadedClassCount")  //返回自 Java 虚拟机开始执行到目前已经加载的类的总数.
```

**ClassLoadingMXBean 能获取的数据:**

getLoadedClassCount()

          返回当前加载到 Java 虚拟机中的类的数量。


getTotalLoadedClassCount()

          返回自 Java 虚拟机开始执行到目前已经加载的类的总数。


getUnloadedClassCount()

          返回自 Java 虚拟机开始执行到目前已经卸载的类的总数。

**MemoryMXBean 能获取的数据:**

gc()

          运行垃圾回收器。


MemoryUsage getHeapMemoryUsage()

          返回用于对象分配的堆的当前内存使用量。


MemoryUsage getNonHeapMemoryUsage()

          返回 Java 虚拟机使用的非堆内存的当前使用量。

ThreadMXBean 能获取的数据:

      getAllThreadIds()
       返回活动线程 ID。


getCurrentThreadCpuTime()

       返回当前线程的总 CPU 时间（以毫微秒为单位）。


getCurrentThreadUserTime()

       返回当前线程在用户模式中执行的 CPU 时间（以毫微秒为单位）。


getDaemonThreadCount()

       返回活动守护线程的当前数目。


getPeakThreadCount()

       返回自从 Java 虚拟机启动或峰值重置以来峰值活动线程计数。


getThreadCount()

       返回活动线程的当前数目，包括守护线程和非守护线程。


getThreadCpuTime(long id)

          返回指定 ID 的线程的总 CPU 时间（以毫微秒为单位）。


ThreadInfo getThreadInfo(long id)

          返回指定 id 的不具有堆栈跟踪的线程的线程信息。



RuntimeMXBean 能获取的数据:
getBootClassPath()  
 返回由引导类加载器用于搜索类文件的引导类路径。  
 String getClassPath()  
 返回系统类加载器用于搜索类文件的 Java 类路径。  
 List<String> getInputArguments()  
 返回传递给 Java 虚拟机的输入变量，其中不包括传递给 main 方法的变量。
String getLibraryPath()  
 返回 Java 库路径。  
 String getManagementSpecVersion()  
 返回正在运行的 Java 虚拟机实现的管理接口的规范版本。  
  
OperatingSystemMXBean 能获取的数据:

getArch()

          返回操作系统的架构。


int getAvailableProcessors()

          返回 Java 虚拟机可以使用的处理器数目。


String getName()

          返回操作系统名称。


String getVersion()

          返回操作系统的版本


MemoryManagerMXBean 能获取的数据:

getMemoryPoolNames()

          返回此内存管理器管理的内存池名称。


String getName()

          返回表示此内存管理器的名称。



还有一些 MXBean 就不全部列出来啦..

我们也可以通过特定的 MBeanServerConnection 远程访问 MXBean:

```java
 JMXServiceURL url = new JMXServiceURL("rmi", "", 0, /jndi/rmi://" + host + ":" + port + "/jmxrmi);
        JMXConnector jmxc = JMXConnectorFactory.connect(url);
        MBeanServerConnection server = jmxc.getMBeanServerConnection();


public static <T> T newPlatformMXBeanProxy(MBeanServerConnection connection,
                                           String mxbeanName,
                                           Class<T> mxbeanInterface)
                                返回用于给定 MXBean 名称的平台 MXBean 接口的代理，以便通过给定 MBeanServerConnection 转发其方法调用。

ClassLoadingMXBean mxClassLoadingMXBean = ManagementFactory.newPlatformMXBeanProxy(mBeanServerConnection, ManagementFactory.CLASS_LOADING_MXBEAN_NAME, ClassLoadingMXBean.class);
CompilationMXBean mxCompilationMXBean = ManagementFactory.newPlatformMXBeanProxy(mBeanServerConnection, ManagementFactory.COMPILATION_MXBEAN_NAME, CompilationMXBean.class);
OperatingSystemMXBean mxOperatingSystemMXBean = ManagementFactory.newPlatformMXBeanProxy(mBeanServerConnection, ManagementFactory.OPERATING_SYSTEM_MXBEAN_NAME, OperatingSystemMXBean.class);
RuntimeMXBean mxRuntimeMXBean = ManagementFactory.newPlatformMXBeanProxy(mBeanServerConnection, ManagementFactory.RUNTIME_MXBEAN_NAME, RuntimeMXBean.class);
GarbageCollectorMXBean mxGarbageCollectorMXBean = ManagementFactory.newPlatformMXBeanProxy(mBeanServerConnection, ManagementFactory.GARBAGE_COLLECTOR_MXBEAN_DOMAIN_TYPE, GarbageCollectorMXBean.class);
MemoryManagerMXBean mxMemoryManagerMXBean = ManagementFactory.newPlatformMXBeanProxy(mBeanServerConnection, ManagementFactory.MEMORY_MANAGER_MXBEAN_DOMAIN_TYPE, MemoryManagerMXBean.class);
MemoryPoolMXBean mxMemoryPoolMXBean = ManagementFactory.newPlatformMXBeanProxy(mBeanServerConnection, ManagementFactory.MEMORY_POOL_MXBEAN_DOMAIN_TYPE, MemoryPoolMXBean.class);
```

使用 MXBean 代理可以方便地远程访问正在运行的虚拟机的平台 MXBean。
所有对 MXBean 代理的方法调用都被转发到 MBeanServerConnection，当连接器服务器出现通信问题时，可能在其中抛出 IOException。
如果使用代理远程访问平台 MXBean 的应用程序要访问 MBeanServerConnector 接口，则应该准备捕获 IOException。
