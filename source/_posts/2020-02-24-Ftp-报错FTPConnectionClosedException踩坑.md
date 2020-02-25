---
title: Ftp 报错FTPConnectionClosedExceptions踩坑
date: 2020-02-24 21:22:36
categories: blog
toc: true
tags:
  - Ftp
  - 问题排查
---

使用过Ftp 存储文件的小伙伴或多或少的肯定遇到过各式各样的Ftp 相关报错，今天我要分享的是我遇到的一个错误FTPConnectionClosedException: Connection closed without indication，其实本身这个错误其实没什么太多可以说的地方，但是这次报这个错误确实有个需要注意的地方，如果没有注意到排查肯定会话费一定的时间，这里分享出来是希望大家不要和我一样继续踩同样的坑。
 
<!--more-->

### 问题

这次报的Ftp 错误是在读取服务器文件的时候抛出的错误，具体错误如下：

```text
Caused by: org.apache.commons.net.ftp.FTPConnectionClosedException: Connection closed without indication.
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:324) ~[commons-net-3.6.jar!/:3.6]
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:300) ~[commons-net-3.6.jar!/:3.6]
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:523) ~[commons-net-3.6.jar!/:3.6]
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:648) ~[commons-net-3.6.jar!/:3.6]
	at org.apache.commons.net.ftp.FTP.sendCommand(FTP.java:622) ~[commons-net-3.6.jar!/:3.6]
	at org.apache.commons.net.ftp.FTP.quit(FTP.java:904) ~[commons-net-3.6.jar!/:3.6]
	at org.apache.commons.net.ftp.FTPClient.logout(FTPClient.java:1148) ~[commons-net-3.6.jar!/:3.6]
	at com.thunisoft.summer.component.store.storage.beanstorage.ftp.AbstractApacheFTPClientProvider.closeFtpClient(AbstractApacheFTPClientProvider.java:200) ~[summer-storage-3.4.0.jar!/:3.4.0]
	at com.thunisoft.summer.component.store.storage.beanstorage.ftp.AbstractApacheFTPClientProvider.getClient(AbstractApacheFTPClientProvider.java:176) ~[summer-storage-3.4.0.jar!/:3.4.0]
	at com.thunisoft.summer.component.store.storage.beanstorage.FTPStorageImpl.getClient(FTPStorageImpl.java:75) ~[summer-storage-3.4.0.jar!/:3.4.0]
	at com.thunisoft.summer.component.store.storage.beanstorage.FTPStorageImpl.getInputStream(FTPStorageImpl.java:114) ~[summer-storage-3.4.0.jar!/:3.4.0]
	... 77 more
```

### 排查经过

#### 分析问题

通过百度可以知道这个问题是FtpClient 获取的连接过早或者被意外关闭的时候抛出IOException，被FtpClient 捕获后抛出的异常。至于为什么连接好端端的会被断开，主要是有下面的几种可能原因：

1.Ftp 服务器连接数已满或者发生了异常导致客户端无法正常获取到连接，这种时候往往Ftp 抛出的异常信息中会包含421错误，类似下面这种：

```text
org.apache.commons.net.ftp.FTPConnectionClosedException: FTP response 421 received.  Server closed connection.
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:388)
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:300)
	at org.apache.commons.net.ftp.FTP._connectAction_(FTP.java:425)
	at org.apache.commons.net.ftp.FTPClient._connectAction_(FTPClient.java:962)
	at org.apache.commons.net.ftp.FTPClient._connectAction_(FTPClient.java:950)
	at org.apache.commons.net.SocketClient._connect(SocketClient.java:244)
	at org.apache.commons.net.SocketClient.connect(SocketClient.java:202)
```

通常高并发的访问Ftp 服务器的时候，由于服务器的单个客户端的连接数配置的比较小的时候容易发生这种错误。

>目前使用场景属于这种情况，需要重点排查！

2.FtpClient 客户端设置的超时时间太短，导致客户端连接被过早的抛弃。

```java
ftpClient.setDataTimeout(60000);       //设置传输超时时间为60秒 
ftpClient.setConnectTimeout(60000);       //连接超时为60秒
```

>这个应该不是导致此次报错的原因，毕竟项目已经使用一段时间了，都是采用的工具类的形式，如果有问题之前就暴露了。

3.获取FtpClient 之后在使用中有不正确的关闭连接的操作，导致连接被意外中断。Apache官网提供的使用FtpClient的代码：

```java
boolean error = false;  
try {  
  int reply;  
  ftp.connect("ftp.foobar.com");  
  System.out.println("Connected to " + server + ".");  
  System.out.print(ftp.getReplyString());  
  
  // After connection attempt, you should check the reply code to verify  
  // success.  
  reply = ftp.getReplyCode();  
  
  if(!FTPReply.isPositiveCompletion(reply)) {  
    ftp.disconnect();  
    System.err.println("FTP server refused connection.");  
    System.exit(1);  
  }  
  ... // transfer files  
  ftp.logout();  
} catch(IOException e) {  
  error = true;  
  e.printStackTrace();  
} finally {  
  if(ftp.isConnected()) {  
    try {  
      ftp.disconnect();  
    } catch(IOException ioe) {  
      // do nothing  
    }  
  }  
  System.exit(error ? 1 : 0);  
}  
```

**特别注意：** 关闭客户端的操作应该在finally 处理。

>此时使用的是公司封装的存储组件，按理说不会有这样的问题，但是有个点让我很在意所以还是需要排查一下！

```java
 public IFTPClient getClient() throws IOException {
        FTPClientCmpt client = new FTPClientCmpt();

        try {
            client.connect(this.getIp(), this.getPort());
            ...

        } catch (Exception var9) {
            this.closeFtpClient(client);
            if (var9 instanceof IOException) {
                throw (IOException) var9;
            }

            throw new IOException(var9);
        }

        return new ApacheFTPClientImpl(client, this.getEncoding());
    }
```

**在意点：** 获取连接后在异常里有一个关闭连接的操作，报错信息也是从closeFtpClient 抛出来的，但是真实错误由于被捕获了看不到，所以还需要确认下。

#### 尝试

1.首先确认Ftp 服务器配置。

现场Ftp 配置如下：

```text
xferlog_enable=YES
#设定开启日志记录功能。
port_enable=YES
connect_from_port_20=YES
dual_log_enable=YES
use_localtime=YES
#设定端口20进行数据连接
xferlog_std_format=YES
#设定日志使用标准的记录格式
listen=YES
#开启独立进程vsftpd，不使用超级进程xinetd。设定该Vsftpd服务工作在StandAlone模式下。
pam_service_name=vsftpd
#设定，启用pam认证，并指定认证文件名/etc/pam.d/vsftpd
userlist_enable=YES
#设定userlist_file中的用户将不得使用FTP
tcp_wrappers=YES
#设定支持TCP Wrappers
chroot_local_user=YES
#限制所有用户在主目录
#限制所有用户在主目录
#以下这些是关于Vsftpd虚拟用户支持的重要配置项目。默认Vsftpd.conf中不包含这些设定项目，需要自己手动添加配置
guest_enable=YES
#设定启用虚拟用户功能
guest_username=ftpuser
#指定虚拟用户的宿主用户
virtual_use_local_privs=YES
#设定虚拟用户的权限符合他们的宿主用户 
user_config_dir=/etc/vsftpd/vuser_conf
#设定虚拟用户个人Vsftp的配置文件存放路径。也就是说，这个被指定的目录里，将存放每个Vsftp虚拟用户个性的配置文件，
#一个需要注意的地方就是这些配置文件名必须和虚拟用户名相同。
local_max_rate=0
```

由于现场采用的vsftpd 虚拟多用户多目录配置的方式，所以还有个用户配置：

```text
local_root=/thunisoft/FtpData
#指定虚拟用户的具体主路径
anonymous_enable=NO
#设定不允许匿名用户访问
write_enable=YES
#设定允许写操作
local_umask=022
#设定上传文件权限掩码
anon_upload_enable=NO
#设定不允许匿名用户上传
anon_mkdir_write_enable=NO
#设定不允许匿名用户建立目录
idle_session_timeout=1200
#设定空闲连接超时时间
data_connection_timeout=300
#设定单次连续传输最大时间
max_clients=300000
#设定并发客户端访问个数
max_per_ip=10000
#设定单个客户端的最大线程数，这个配置主要来照顾Flashget、迅雷等多线程下载软件
local_max_rate=0
#设定该用户的最大传输速率，单位b/s
```

>**注意：** 这个是本次要将的坑点，多用户配置！

和客户端相关联的几个配置主要是：

1.idle_session_timeout --设定空闲连接超时时间
2.data_connection_timeout --设定单次连续传输最大时间
3.max_clients --设定并发客户端访问个数
4.max_per_ip --设定单个客户端的最大线程数，这个配置主要来照顾Flashget、迅雷等多线程下载软件

这几个关键参数都设置的大的离谱，在现场环境根本就不需要这么大，但是按理说也不会有问题，但是保险起见还是让现场运维修改了配置：

```text
idle_session_timeout=600
max_per_ip=100
max_clients=200
```

修改完配置后，通过命令重新加载配置后发现问题还是存在，同样报错。

```text
//For SysVinit Systems
service vsftpd reload

//For systemd Systems
systemctl reload vsftpd
```

通过命令
`netstat -na|grep 21|grep ESTABLISHED|awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -r`
查看ftp 客户端连接情况，发现单个客户端的连接并不高，也就80-90的样子，所以配置的单个客户端的并发数100应该不会有问题才对（<span style="color: red">这里有疑问！</span>）。

2.确认现场网络环境

现场确认通过工具或者浏览器直接访问Ftp 服务，发现能够正常访问，可以确定网络环境应该是没什么问题的，但还是不放心是不是防火墙的原因，所以又找现场确认了防火墙的情况，发现现场根本没有开防火墙，由此可以肯定Ftp 服务网络是没有问题的。

Ftp 相关的命令：

```text
firewall-cmd --state                           ##查看防火墙状态，是否是running
firewall-cmd --reload                          ##重新载入配置，比如添加规则之后，需要执行此命令
firewall-cmd --get-zones                       ##列出支持的zone
firewall-cmd --get-services                    ##列出支持的服务，在列表中的服务是放行的
firewall-cmd --query-service ftp               ##查看ftp服务是否支持，返回yes或者no
firewall-cmd --add-service=ftp                 ##临时开放ftp服务
firewall-cmd --add-service=ftp --permanent     ##永久开放ftp服务
firewall-cmd --remove-service=ftp --permanent  ##永久移除ftp服务
firewall-cmd --add-port=80/tcp --permanent     ##永久添加80端口 
firewall-cmd --remove-port=80/tcp --permanent  ##永久移除80端口
firewall-cmd --list-ports                      ##查看已经开放的端口
iptables -L -n                                 ##查看规则，这个命令是和iptables的相同的
man firewall-cmd                               ##查看帮助
```

3.检查代码。

通过抛出的问题的代码行数去追查了使用Ftp 的业务代码，经过调试发现业务逻辑是没有问题唯一在意的地方在于存储服务获取FtpClient 连接的地方，由于代码在jar 包里，没办法为了确认问题只有重写源码的方式再次调试，
经过调试发现最终抛出的异常是：

```java
org.apache.commons.net.ftp.FTPConnectionClosedException: FTP response 421 received.  Server closed connection.
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:388)
	at org.apache.commons.net.ftp.FTP.__getReply(FTP.java:300)
	...
	
```

抛出的421 错误应该还是由于连接数满了导致的，所以基本肯定问题不在代码逻辑还是出在Ftp 服务器。

4.查看Ftp 服务的相关日志

由于之前一直没有获取到现场的Ftp 日志，所以之前的排查都是基于没有日志的情况，但是其实首先应该去排查Ftp 日志，这个需要注意。

```text
Sat Feb 22 13:30:20 2020 [pid 2870] CONNECT: Client "151.15.1.50"
Sat Feb 22 13:30:20 2020 [pid 2871] CONNECT: Client "151.15.1.50", "Connection refused: too many sessions for this address."
Sat Feb 22 13:30:20 2020 [pid 2868] [dzdaftp] OK LOGIN: Client "151.15.1.50"
Sat Feb 22 13:30:20 2020 [pid 2873] [dzdaftp] OK DOWNLOAD: Client "151.15.1.50", "/EAMS_3283/2020/2/c8d28c8e75cf479da2d14eb23e5a3efc/07742ff15b3e40a9982f54d8576fc44a.jpg", 487172 bytes, 14704.19Kbyte/sec
Sat Feb 22 13:30:21 2020 [pid 2880] CONNECT: Client "151.15.1.50", "Connection refused: too many sessions for this address."
Sat Feb 22 13:30:21 2020 [pid 2881] CONNECT: Client "151.15.1.50", "Connection refused: too many sessions for this address."
Sat Feb 22 13:30:21 2020 [pid 2884] CONNECT: Client "151.15.1.50", "Connection refused: too many sessions for this address."
Sat Feb 22 13:30:21 2020 [pid 2885] CONNECT: Client "151.15.1.50", "Connection refused: too many sessions for this address."
Sat Feb 22 13:30:21 2020 [pid 2888] CONNECT: Client "151.15.1.50", "Connection refused: too many sessions for this address."
Sat Feb 22 13:30:21 2020 [pid 2889] CONNECT: Client "151.15.1.50", "Connection refused: too many sessions for this address."
Sat Feb 22 13:30:21 2020 [pid 2892] CONNECT: Client "151.15.1.50", "Connection refused: too many sessions for this address."
Sat Feb 22 13:30:21 2020 [pid 2893] CONNECT: Client "151.15.1.50", "Connection refused: too many sessions for this address."
...

```

可以从日志里看到单个客户端并发连接Ftp 服务的时候疯狂的报`Connection refused: too many sessions for this address`错误，这里基本可以肯定还是Ftp 配置的问题。

### 处理问题

既然已经确定问题出在Ftp 配置上，没有别的办法只有从配置上想办法，通过不停的修改配置文件发现一个奇怪的现象，通过命令`netstat -apn | grep ftp | wc -l`查看Ftp 连接数的情况的时候，发现报错的时候总是出现在连接数超过50的时候，这里让我产生了怀疑，众所周知Ftp 服务器的默认连接数就是50，难道一直修改的配置不对（这里一直是修改的用户配置），其实一直走的是默认配置嘛？

通过命令`ps -ef | grep ftp`查看Ftp 的配置使用情况：

![Ftp 配置使用情况](/assets/img/ftpConfig.png)

发现确实Ftp 服务器一直都是走的默认配置，修改了vsftpd.conf 的配置后发现问题解决了。

后来查询资料发现，vsftpd.conf 文档有如下说明：

```text
user_config_dir：
This powerful option allows the override of any config option specified in the manual page, 
on a per-user basis. Usage is simple, and is best illustrated with an example. If you set 
user_config_dir to be /etc/vsftpd/user_conf and then log on as the user "chris", 
then vsftpd will apply the settings in the file /etc/vsftpd/user_conf/chris for the duration of the session. 
The format of this file is as detailed in this manual page! 
PLEASE NOTE that not all settings are effective on a per-user basis. 
For example, many settings only prior to the user's session being started. 
Examples of settings which will not affect any behviour on a per-user basis include listen_address, 
banner_file, max_per_ip, max_clients, xferlog_file, etc.
```

中文意思：
这个选项允许基于每个用户重写任何配置选项。使用非常简单，最好阐述方法就是给出例子。如果你设置user_config_dir作为/etc/vsftpd_user_conf,然后登录的用户名为"chris"，那么vsftpd应用/etc/vsftpd_user_conf/chris配置文件这个文件格式就是本手册的阐述的，注意：并不是所有的设置都是基于用户的。例如，许多设置只能用户会话开始。例如一些设置不是基于用户的，例如listen_address, banner_file，max_per_ip, max_clients, xferlog_file等等。


所以在多用户配置的情况下有些配置在用户配置文件中修改并不生效，特别是这次问题的关键`max_per_ip, max_clients`，这里确实有点坑，如果不熟悉配置的使用要求的话根本不知道为什么在用户配置中做了设置却不生效。

### 总结

这个问题排查其实花了不少时间，其实问题比较简单，但是如果不清楚问题的关键——就是这次说得多用户配置下有些配置并不会生效，还是会和我一样花时间排查，所以这里分享给大家，希望大家不需要踩同样的坑。另外感觉现场运维同学如果不清楚配置含义还是应该使用默认配置的方式部署Ftp ，这样也不会出现类似的问题。总的来说就是不清楚的按官方推荐的做就可以了！