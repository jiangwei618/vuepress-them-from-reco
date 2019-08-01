#UbuntuLinux系统环境变量配置文件
&emsp;Ubuntu Linux系统环境变量配置文件分为两种：系统级文件和用户级文件，下面详细介绍环境变量的配置文件。 

##系统级文件：
&emsp;/etc/profile:  
在登录时,操作系统定制用户环境时使用的第一个文件，此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行。并从/etc/profile.d目录的配置文件中搜集shell的设置。这个文件一般就是调用/etc/bash.bashrc文件。  

&emsp;/etc/bash.bashrc：  
系统级的bashrc文件，为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.  

&emsp;/etc/environment:  
在登录时操作系统使用的第二个文件,系统在读取你自己的profile前,设置环境文件的环境变量。  

##用户级文件：
&emsp;~/.profile:  
在登录时用到的第三个文件 是.profile文件,每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件。  

&emsp;~/.bashrc:  
该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取。不推荐放到这儿，因为每开一个shell，这个文件会读取一次，效率 上讲不好。  

&emsp;~/.bash_profile：  
每个用户都可使用该文件输入专用于自己 使用的shell信息,当用户登录时,该文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件。~/.bash_profile 是交互式、login 方式进入 bash 运行的~/&emsp;.bashrc是交互式 non-login 方式进入 bash 运行的通常二者设置大致相同，所以通常前者会调用后者。  

&emsp;~./bash_login:  
不推荐使用这个，这些不会影响图形界面。而且.bash_profile优先级比bash_login高。当它们存在时，登录shell启动时会读取它们。    

&emsp;~/.bash_logout:  
当每次退出系统(退出bash shell)时,执行该文件.  

&emsp;~/.pam_environment：用户级的环境变量设置文件。  
&emsp;另外,/etc/profile中设定的变量(全局)的可以作用于任何用户,而~/.bashrc等中设定的变量(局部)只能继承 /etc/profile中的变量,他们是"父子"关系。   

##/etc/profile与/etc/enviroment的比较
&emsp;/etc/profile 是所有用户的环境变量  

&emsp;/etc/enviroment是系统的环境变量  

&emsp;如果同一个变量在用户环境(/etc/profile)和系统环境(/etc/environment) 有不同的值那应该是以用户环境为准了。  


##设置环境变量的方法
由以上分析可知：  
&emsp;/etc/profile全局的，随系统启动设置【设置这个文件是一劳永逸的办法】  
&emsp;/root/.profile和/home/myname/.profile只对当前窗口有效。  
&emsp;/root/.bashrc和 /home/yourname/.bashrc随系统启动，设置用户的环境变量【平时设置这个文件就可以了】  
那么要配置Ubuntu的环境变量，就是在这几个配置文件中找一个合适的文件进行操作了；如想将一个路径加入到$PATH中，可以由下面这样几种添加方法：  
1.控制台中：  
`$PATH="$PATH:/my_new_path"    （关闭shell，会还原PATH）`  
2.修改profile文件：  
```
$sudo gedit /etc/profile
在里面加入:
exportPATH="$PATH:/my_new_path"

```  
3.修改.bashrc文件：
```
$ sudo gedit /root/.bashrc
在里面加入：
export PATH="$PATH:/my_new_path"
```  
**后两种方法一般需要重新注销系统才能生效，最后可以通过echo命令测试一下：**  

四、小结
&emsp;综上所述，在Ubuntu 系统中/etc/profile文件是全局的环境变量配置文件，它适用于所有的shell。在我们登陆Linux系统时，首先启动/etc/profile文件，然后再启动用户目录下的~/.bash_profile、~/.bash_login或~/.profile文件中的其中一个，执行的顺序和上面的排序一样。如果~/.bash_profile文件存在的话，一般还会执行~/.bashrc文件。