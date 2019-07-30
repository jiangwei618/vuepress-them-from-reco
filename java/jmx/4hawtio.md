## 1.github 地址
https://github.com/hawtio/hawtio/tree/master/hawtio-app

## 2.使用方法

1. 从http://hawt.io/getstarted/index.html下载最新版本的hawtio的最新运行jar包hawtio-app-1.4.66.jar 下载好后，切换到jar包所在目录 执行    

1. java -jar hawtio-app-1.4.66.jar
     启动hawtio程序，默认程序占用端口号为8080,如果8080端口已经被占用，请使用port启动参数，指定启动端口号，例如，我们使用8090作为启动端口号,启动hawtio程序

1. java -jar hawtio-app-1.4.66.jar -–port 8090

&emsp;&emsp;当浏览器出现hawtio的欢迎页面(http://[hawtio所在主机ip]:[hawtio启动端口号)/hawtio/welcome)后，表示hawtio程序启动成功，我们可以使用它连接远程ActiveMQ服务器

&emsp;&emsp;在欢迎界面中点击上侧菜单条中的[Connect]菜单，打开Connect界面，在这个页面我们可以配置和连接我们想要连接的服务器。

- 项目连接名称(用于保存连接使用)
 
- 主机ip(或主机名)

- 端口号(即jetty.xml文件中配置的管理页面端口号)

- jolokia路径名(对ActiveMQ，使用api/jolokia)（odl：/jolokia）

&emsp;&emsp;配置完成后，可以点击[Save]按钮进行保存，下次启动hawtio时即可以从下拉列表中选择保存的连接进行连接。

&emsp;&emsp;**点击[Connect to remote server]，连接远端的ActiveMQ服务器，连接服务器时会要求你输入连接用户名和密码(即ActiveMQ的管理页面登录用户名/密码)，输入完成，验证通过后，即连接服务器成功。**

## 3.修改源代码以实现远程部署
 1：修改类.
 io.hawt.system.ProxyWhitelist
 ```
  public boolean isAllowed(ProxyDetails details) {
        return true;
    }
 ```
 2：修改web.xml
\hawtio\hawtio-base\src\main\webapp\WEB-INF\web.xml
```
 <init-param>
      <param-name>proxyWhitelist</param-name>
      <param-value>
        localhost,
        127.0.0.1
      </param-value>
    </init-param>
```
3:启动jar时指定外部web.xml
