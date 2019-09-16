---
title： osgi service
date： 2019-08-26 11：11：19
description： osgi service
categories：
- 框架
- odl
- osgi_karaf
tags：
- service
- osgi
export_on_save：
markdown： true
---

## 1. 服务获取工具类

### 1.1. WaitingServiceTracker

```java
 final WaitingServiceTracker<MyService> tracker = WaitingServiceTracker.create(MyService.class, bundleContext);
 final MyService myService = tracker.waitForService(WaitingServiceTracker.FIVE_MINUTES);
```

### 1.2. ServiceTracker

```java
BundleContext bundleContext = FrameworkUtil.getBundle(Util.class).getBundleContext();
ServiceTracker serviceTracker = new ServiceTracker<>(bundleContext, service, null);serviceTracker.open();
S object = null;
try {
    object = (S) serviceTracker.waitForService(TIME_OUT);
} catch (InterruptedException e) {
    LOG.error("Error to get service {} , {}", service, e);
}return object;
```

## 2. 发布服务并关联属性。

```xml
 <service ref="beanToBeExported" interface="com.xyz.MyServiceInterface">
     <service-properties>
         <beans：entry key="myOtherKey" value="aStringValue"/>
         <beans：entry key="aThirdKey" value-ref="beanToExposeAsProperty"/>
     </service-properties>
 </service>
```

## 3. blueprint 获取服务

每个以 Spring Bean 发布成服务都会有一个属性名为 org.springframework.osgi.bean.name，对应的值为目标 Bean 的 name。

```xml
 <service ref="beanToPublish" interface="com.xyz.MessageService"/>
```

这个服务就会有一个属性 org.springframework.osgi.bean.name，值为 beanToPublish。

## 4. 服务作用域

Spring DM 引入一种 Bean 的作用域，叫 bundle scope。当导出服务的 Bean 加了这个作用域后，导入这个服务的 Bundle 会创建一个新的服务 Bean 实例。

```xml
 <service ref="beanToBeExported" interface="com.xyz.MessageService"/>
 <bean id="beanToBeExported" scope="bundle" class="com.xyz.MessageServiceImpl"/>
```

## 5. 服务多个接口导出

当有一个 Bean 服务存在多个接口时，需要将接口都导出：

```xml
 <service ref="beanToBeExported">
 　　<interfaces>
 　　　　<value>com.xyz.MessageService</value>
 　　　　<value>com.xyz.MarkerInterface</value>
 　　</interfaces>
 </osgi：service>
 or
 <service ref="beanToBeExported" auto-export="interfaces"/>
```

<strong> auto-export 有四种值： </strong>

disabled：如果 auto-export 属性未被指定，则该选项为默认值。接口列表必须使用 interface 属性或 interfaces 子元素指定。
interfaces：使用由服务类或其任何超类实现的所有公共接口注册服务。
class-hierarchy：使用服务类或其任何公共超类注册服务。
all-classes：结合 interfaces 和 class-hierarchy 选项。

<strong> 服务依赖属性 </strong>
depends-on：

```xml
 <service ref="beanToBeExported" interface="com.xyz.MyServiceInterface" depends-on="myOtherComponent"/>
```

配置 beanToBeExported 服务所依赖的组件 myOtherComponent 初始化。

<strong> context-class-loader： </strong>

用于配置使用第三方的 classloader 来加载服务。

<strong> ranking： </strong>

默放为 0，如果存在多个可用的服务接口，那么返回具有最大的 ranking 值的服务。如果 ranking 值相同，刚返回 service id 小的那个。

```xml
 <service ref="beanToBeExported" interface="com.xyz.MyServiceInterface"  ranking="9"/>
```

## 6. 多接口服务引用

```xml
 <reference id="importedOsgiService">
 　　<interfaces>
 　　　　<value>com.xyz.MessageService</value>
 　　　　<value>com.xyz.MarkerInterface</value>
 　　</interfaces>
 </reference>
```

这个 reference 的 Bean 是实现了 MessageService，MarkerInterface 接口的。

引用服务的一些属性

<strong> filter 属性 </strong>

```xml
 <reference id="asyncMessageService" interface="com.xyz.MessageService" filter="(asynchronous-delivery=true)"/>
```

过滤服务属性 asynchronous-delivery 为 true 的服务。

<strong> bean-name 属性 </strong>

```xml
 <osgi：reference id="messageService" interface="com.xyz.MessageService" bean-name="messageServiceBean"/>
```

返回服务 Bean 的 Id 为 messageServiceBean 的服务。

<strong> cardinality 属性 </strong>

```xml
 <osgi：reference id="messageService" interface="com.xyz.MessageService" cardinality="1..1"/>
```

默认值为 1..1，说明总是有一个这样的服务存在。0..1 说明不需要一直存在这样一个服务。

<strong> depends-on： </strong>

当 depends-on 属性 Bean 实例化后，这个服务才能查找该服务。

<strong> context-class-loader： </strong>

与上对应。如果两边都有，最后启作用的是 exporter 那端的。

<strong> timeout： </strong>

等待服务多少秒，默认 300 秒。

<strong> List，set 引用服务支持属性 </strong>

interface 　　 filter 　　 bean-name 　　 cardinality 　　 context-class-loader

cardinality 取值为 0..N，1..N，前面说明可以不存在，后面说明至少存在一个。

<strong> 注意： </strong>当一个 Bundle 暴露 SubInterface 接口服务，而另一个 Bundle 引入 SuperInterface 接口是不匹配的。

当注册的服务中有接口，类等，能通过 greedy-proxying 创建代理访问导入服务中的的所有包含在该 Bundle 中类。

```java
 list id="services" interface="com.xyz.SomeService" greedy-proxying="true"/>
 for (Iterator iterator = services.iterator(); iterator.hasNext();) {
 　　SomeService service = (SomeService) iterator.next();
 　　service.executeOperation();
 　　// if the service implements an additional type
 　　// do something extra
 　　if (service instanceof MessageDispatcher) {
 　　　　((MessageDispatcher)service).sendAckMessage();
 　　}
 }
```

## 7. 服务监听 Service Listener：

```xml
 <service ref="beanToBeExported" interface="SomeInterface">

    <registration-listener ref="myListener" registration-method="serviceRegistered" unregistration-method="serviceUnregistered"/>　　

    <registration-listener registration-method="register">　　　　
        <bean class="SomeListenerClass"/>　　
    </registration-listener>
 </service>
```

```java
 public void anyMethodName(ServiceType serviceInstance, Map serviceProperties);
 public void anyMethodName(ServiceType serviceInstance, Dictionary serviceProperties);
```

ServiceType 为服务接口 interface，serviceProperties 保存着这个服务的所有属性，兼容性考虑的话，可以选 Dictionary。
有另一种方法，不鼓励这样用，那就是实现 Spring DM 特定接口 OsgiServiceRegistrationListener，这样做的好处是省去了声明 registration-method，unregistration-method，坏处是你的类与 Spring 有引用关系，与 JAVA 简单 POJO 类编程。

<strong> Dealing With The Dynamics Of OSGi Imported Services </strong>
An example of declaring a listener that implements OsgiServiceLifecycleListener：

```xml
 <reference id="someService" interface="com.xyz.MessageService">
     <listener ref="aListenerBean"/>
 </reference>
```

An example of declaring an inline listener bean with custom bind and unbind methods：

```xml
 <reference id="someService" interface="com.xyz.MessageService">
     <listener bind-method="onBind" unbind-method="onUnbind">
         <beans:bean class="MyCustomListener"/>
     </listener>
 </reference>
```

<strong> Listener Attributes： </strong>

ref 　　 bind-method 　　 unbind-mehtod

<strong> Listener And Service Proxies： </strong>

Spring 管理这个服务，但是调用这个服务的时候实质是一个代理。这样做的原因是防止 Listener 持有一个服务的强引用。Lisenter 感兴趣的是监听服务而不是依赖对等实例，更加关注服务接口，而不是他的身份，服务属性和服务跟踪。

## 8. 服务引用可以排序

```xml
 <set id="myServices" interface="com.xyz.MyService" comparator-ref="someComparator"/>
     <list id="myOtherServices" interface="com.xyz.OtherService">
     <comparator>
         <beans:bean class="MyOtherServiceComparator"/>
     </comparator>
 </list>
```

Comparator-ref 是一个实现了 java.util.Comparator 接口的 Bean。

```xml
 <list id="myServices" interface="com.xyz.MyService">
 　　<comparator><natural basis="services"/></comparator>
 </list>
 <set id="myOtherServices"interface="com.xyz.OtherService">
 　　<comparator><natural basis="service-references"/></comparator>
 </set>
```

默认提供的一些排序，详细可以去了解下。

服务最佳实践

1.  在 listener 中不要执行太长的活动，因为这个方法是同步的，太长将会影响这个 listener 监听其他事件。
2.  创建自己的 listener，不要引用 Spring DM 的 API。
3.  如果 listener 重复声明 bind/unbind 方法，可以考虑写一个通用的可重用的 Bean。
4.  服务属性优先 java.util.Map 而不是 java.util.Dictionary。
5.  重载方法要小心，因为当服务类型匹配方法服务类型的时候，该方法就会执行

Object type - will match all services for which the listener is triggered. This method will be always called.
Collection type - if this method is called, the Object method is also called.
SortedSet type - if this method is called, then both the Object and Collection methods are called.

Service Importer Global Defaults

```xml
 default-timeout
 <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:osgi="http://www.springframework.org/schema/osgi"
          osgi:default-timeout="5000">
      <reference id="someService" interface="com.xyz.AService"/>
      <reference id="someOtherService" interface="com.xyz.BService" timeout="1000"/></beans:beans>

 default-cardinality
 <beans:beans
          xmlns="http://www.springframework.org/schema/osgi"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:beans="http://www.springframework.org/schema/beans"
          xmlns:osgi="http://www.springframework.org/schema/osgi"
          osgi:default-cardinality="0..X"
          default-lazy-init="false">

 　　<reference id="someService" interface="com.xyz.AService"/>　　<set id="someSetOfService" interface="com.xyz.BService"/>　　<list id="anotherListOfServices" interface="com.xyz.CService" cardinality="1..N"/>
 </beans:beans>
```
