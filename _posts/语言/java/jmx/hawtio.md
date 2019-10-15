---
title: hawtio
date: 2019-08-26 10:57:34
description: hawtio
categories:
  - 语言
  - java
  - jmx
tags:
  - hawtio
  - jmx


---

## 1.github 地址

[github 地址](https://github.com/hawtio/hawtio/tree/master/hawtio-app)
[修改后的 jar](https://github.com/jiangwei618/note/blob/master/%E8%AF%AD%E8%A8%80/java/jmx/hawtio-app-2.7-SNAPSHOT.jar)

## 2.使用方法

1. 从http://hawt.io/getstarted/index.html下载最新版本的hawtio的最新运行jar包hawtio-app-1.4.66.jar 下载好后，切换到 jar 包所在目录 执行

1. java -jar hawtio-app-1.4.66.jar
   启动 hawtio 程序，默认程序占用端口号为 8080,如果 8080 端口已经被占用，请使用 port 启动参数，指定启动端口号，例如，我们使用 8090 作为启动端口号,启动 hawtio 程序

1. java -jar hawtio-app-1.4.66.jar -–port 8090

&emsp;&emsp;当浏览器出现 hawtio 的欢迎页面(http://[hawtio所在主机ip]:[hawtio启动端口号)/hawtio/welcome)后，表示 hawtio 程序启动成功，我们可以使用它连接远程 ActiveMQ 服务器

&emsp;&emsp;在欢迎界面中点击上侧菜单条中的[Connect]菜单，打开 Connect 界面，在这个页面我们可以配置和连接我们想要连接的服务器。

- 项目连接名称(用于保存连接使用)

- 主机 ip(或主机名)

- 端口号(即 jetty.xml 文件中配置的管理页面端口号)

- jolokia 路径名(对 ActiveMQ，使用 api/jolokia)（odl：/jolokia）

&emsp;&emsp;配置完成后，可以点击[Save]按钮进行保存，下次启动 hawtio 时即可以从下拉列表中选择保存的连接进行连接。

&emsp;&emsp;**点击[Connect to remote server]，连接远端的 ActiveMQ 服务器，连接服务器时会要求你输入连接用户名和密码(即 ActiveMQ 的管理页面登录用户名/密码)，输入完成，验证通过后，即连接服务器成功。**

## 3.修改源代码以实现远程部署

1：修改类.
io.hawt.system.ProxyWhitelist

```java
 public boolean isAllowed(ProxyDetails details) {
       return true;
   }
```

2：修改 web.xml
\hawtio\hawtio-base\src\main\webapp\WEB-INF\web.xml

```xml
 <init-param>
      <param-name>proxyWhitelist</param-name>
      <param-value>
        localhost,
        127.0.0.1
      </param-value>
    </init-param>
```

3:启动 jar 时指定外部 web.xml
