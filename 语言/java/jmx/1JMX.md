# Jmx介绍
## 1.简介
&emsp;&emsp;全称Java Management Extensions，从Java5.0开始引入到标准Java技术平台中。JMX提供了一个标准的方法去管理资源，因为JMX是一种动态技术，你可以在被管理资源创建、实例化和实现的时候监控和管理他们。你也可以使用JMX技术去监听和管理Java虚拟机。

## 2.Why JMX
&emsp;&emsp;JMX技术使Java应用程序无需在管理方面投入大量精力
JMX几乎可以在任何支持Java的设备上运行，而且只需要嵌入一个管理对象服务器（MBean Server），使它的一些功能作为一个或几个被管理的bean(MBean)在目标服务器上注册。然后就可以从管理基础设施中受益。

&emsp;&emsp;JMX技术提供了一个标准的方法去管理Java应用、系统和网络
标准Java平台都支持JMX架构，因此只要符合JMX规范都可以通过JMX管理。

&emsp;&emsp;JMX让为Java应用提供了可伸缩、动态、分布式的管理架构
使Java应用可以远程管理。

## 3.JMX架构

![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/1JMX.md-2019-08-06-14-59-25.png)

---

![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/1JMX.md-2019-08-06-15-00-01.png)


## 4.基本术语
&emsp;&emsp;Instrumentation
要管理的资源。使用Java Bean描述要管理的资源。这些Java Bean叫MBean（Management Bean）。

&emsp;&emsp;MBean Agent
代理层。主要定义了各种服务以及通信模型，和需要被管理的资源在同一机器上，核心模块是MBean Server，所有的MBean都要向它注册，才能被管理。注册在MBeanServer上的MBean并不直接和远程应用程序进行通信，他们通过协议适配器（Adapter）和连接器（Connector）进行通信。

&emsp;&emsp;Distributed Layer
也叫Remote Management Layer. 即远程管理层。MBean Server依赖于该层的协议适配器（Adaptor）和连接器（Connector），让JMX Agent可以被该JVM外面的管理系统远程访问。支持多种协议：SNMP，HTML，RMI.

## 5.MBean在JDK中的应用
&emsp;&emsp;代码逻辑主要在java.lang.management包中
![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/1JMX.md-2019-08-06-15-00-09.png)


平台资源 | 对应的 MXBean | 可使用的数量
---|--- | ---
缓冲池|	BufferPoolMXBean|	1个或多个
类装入系统|	ClassLoadingMXBean|	1个
编译系统|	CompilationMXBean|	1个
VM HotSpotDiagnosticMXBean	|垃圾收集系统	GarbageCollectorMXBean |至少 1
内存管理器|	MemoryManagerMXBean|	1个或多个
内存|	MemoryMXBean|	1个
内存资源|	MemoryPoolMXBean|	1个
操作系统|	OperatingSystemMXBean|	1个
logging	| PlatformLoggingMXBean|	1个
运行时系统|	RuntimeMXBean	|1个
系统资源压力|	SystemResourcePressureMXBean
线程|	ThreadMXBean|	1个
JDK中提供的这些MBean可以通过ManagementFactory获取实例。


## 6.MBean分类
&emsp;&emsp;Standard MBean
&emsp;&emsp;MXBean
&emsp;&emsp;Dynamic MBean
&emsp;&emsp;Open MBean
&emsp;&emsp;Module MBean

### 6.1 MBean规范
- A set of readable or writable attributes, or both.
- A set of invokable operations.
- A self-description.
- 管理接口贯穿于MBean的整个生命周期，并且是不变的。Mbean在某些预先定义的事件发生时可以发出通知（Notifications）。

### 6.2 Standard MBean
标准MBean，需要满足规范：

- 定义管理接口SomethingMBean，并且有一个叫Something的实现类。**标准MBean是这二者的组合，而且这两个必须在一个包下。**
例子：
```
package com.example; 
public interface HelloMBean { 
 
    public void sayHello(); 
    public int add(int x, int y); 
    
    public String getName(); 
     
    public int getCacheSize(); 
    public void setCacheSize(int size); 
} 

```
```
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
&emsp;&emsp;与标准MBean类似，MXBean接口进行自述，以及一个实现类。
不同于标准MBean，实现类可以叫任意名字。
### 6.4 Dynamic MBean
&emsp;&emsp;动态MBean即编码方式实现的MBean。

&emsp;&emsp;不再通过定义接口来进行自述，而是通过实现DynamicMBean，定义metadata来进行描述。

```
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

  * MBeanInfo描述管理接口
  * Attribute的getter和setter提供通用属性设置，定义对外暴露的属性
  * invoke提供通用操作，定义对外暴露的操作
  * 注意：
    * Dynamic的意思是管理接口在运行时才会真正的显现出来，不用通过事先静态的接口定义。而不是指可以动态的调整管理接口。
    * MBean的描述应该是不会改变的，所以一般要在动态MBean的构造函数里来构建MBeanInfo。

### 6.5 Module MBean
&emsp;&emsp;Module MBean是动态MBean的一种实现，所以它的接口、属性、操作也是通过编程方式来定义。

&emsp;&emsp;JMX规范规定该类必须实现为javax.management.modelmbean.RequiredModelMBean，管理者要做的就是实例化该类，并配置该构件的默认行为并注册到JMX代理中，即可实现对资源的管理。RequiredModelMBean是一个没有任何管理接口的动态MBean，但是它可以把MBeanInfo跟一个目标对象组合起来。

&emsp;&emsp;目标对象是具体实现管理行为的类。通过ModelMBean接口的setManagedResource()可以将ModelMBean和目标对象进行关联。

  ```java
public void setManagedResource(Object managedResource, String managedResourceType) ;
```
&emsp;&emsp;managedResourceType的值可以为ObjectReference, Handle, IOR, EJBHandle或RMIReference，但当前只支持ObjectReference.

![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/1JMX.md-2019-08-06-15-00-32.png)

&emsp;&emsp;Module MBean具有以下新的特点：
- 持久性。定义了持久机制，可以利用Java的序列化或JDBC来存储模型MBean的状态。
- 通知和日志功能。能记录每一个发出的通知，并能自动发出属性变化通知。
- 属性值缓存。具有缓存属性值的能力。

&emsp;&emsp;使用Apache commons-modeler 简化Module MBean的开发
commons-modeler支持xml描述管理接口，使用动态MBean可以配置式开发
参考：http://blog.csdn.net/s464036801/article/details/9980439

## 6.5 Open MBean
&emsp;&emsp;也是一种动态MBean，规范还在完善中

## 7.Notifications
&emsp;&emsp;MBean提供了一套通知机制，其实就是观察者模式
继承javax.management.NotificationBroadcasterSupport就可以发出通知
实现javax.management.NotificationListener可以接收通知

## 8.Agent
&emsp;&emsp;Agent作为中间代理层，管理着MBeans并且对外暴露管理接口，核心模块MBean Server
通过工厂类：java.lang.management.ManagementFactory可以创建MBean Server，然后在server上注册MBean。
如果是只有本地使用，注册完后就可以通过JConsole连接管理了。
如果需要远程管理，则需要配置协议适配器提供远程管理能力。

## 9.Remote Management
&emsp;&emsp;远程接口的暴露
通过com.sun.jdmk.comm.HtmlAdaptorServer暴露HTML远程管理接口
通过javax.management.remote.JMXConnectorServer暴露RMI远程管理接口
```
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
&emsp;&emsp;HTML方式的远程客户端就是浏览器

&emsp;&emsp;RMI方式可以使管理客户端，如JConsole，也可以是自己编写的Client
项目中的应用

&emsp;&emsp;Spring JMX
http://blog.csdn.net/yaerfeng/article/details/28232435

&emsp;&emsp;log4j JMX
http://logging.apache.org/log4j/2.x/manual/jmx.html

&emsp;&emsp;Tomcat JMX
http://tomcat.apache.org/tomcat-6.0-doc/monitoring.html
