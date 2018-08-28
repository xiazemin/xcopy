# Tcpcopy简介

TCPCopy是一种请求复制（所有基于tcp的packets）工具 ，其功能是复制在线数据包，修改TCP/IP头部信息，发送给测试服务器，达到欺骗测试服务器的TCP 程序的目的，从而为欺骗上层应用打下坚实基础。



TCPCopy七大功能

1）分布式压力测试工具，利用在线数据，可以测试系统能够承受的压力大小（远比ab压力测试工具真实地多）,也可以提前发现一些bug

2）普通上线测试，可以发现新系统是否稳定，提前发现上线过程中会出现的诸多问题，让开发者有信心上线

3）对比试验，同样请求，针对不同或不同版本程序，可以做性能对比等试验

4）流量放大功能，可以利用多种手段构造无限在线压力，满足中小网站压力测试要求

5）利用TCPCopy转发传统压力测试工具发出的请求，可以增加网络延迟，使其压力测试更加真实

6）热备份

7）实战演习（架构师必备）

TCPCopy分为TCPCopy client和TCPCopy server

其中TCPCopy client运行在在线服务器上面，用来捕获在线请求数据包；TCPCopy server（监听端口为36524）运行在测试机器上面，在测试服务器的响应包丢弃之前截获测试服务器的响应包，并通过TCPCopy client和TCPCopy server之间的tcp连接传递响应包的tcp和ip

头部信息给TCPCopy client，以完成TCP交互。



启动tcpcopy

TCPCopy server （root用户执行）

1）启动内核模块ip\_queue



\#modprobe ip\_queue

2）设置要截获的端口，并且设置对output截获



\#iptables -I OUTPUT -p tcp --sport &lt;port&gt; -j QUEUE 

3）启动intercept



intercept

注意：

1.如果已经启动ip\_queue和已经设置iptables，只需要运行第3项;



iptables --list





2.测试完以后要记得删除上面设置的iptables条目

清空iptables：



iptables -F

3.为了避免不必要的麻烦，关闭的时候先关闭tcpcopy，然后再关闭intercept

TCPCopy client （root用户执行）

tcpcopy 0.6版本

./tcpcopy -x 服务器应用端口号-测试服务器ip地址：测试服务器应用端口

-n 参数

进行多重复制，此参数的值就是代表复制过去的流量是在线的n 倍

其他参数请参看文档



Tcpcopy实战

我测试的项目是一个基于RFID的物联网采集项目，采集的主要功能是接收基站发过来的TCP数据对其进行解析，分发。当时选工具的时候也考虑了好几个，Jmeter好像没有这方面功能，而LoadRunner的windows sockets又相对复杂，所以选择了简单易用的tcpcopy。



测试环境

2台ubuntu/linux机器，一台作为在线服务器，用来接收真实的基站信息，一台用来做测试服务器，用来承受在线服务器流量翻倍后的压力。在线服务器使用4001端口，为了保持一致，测试服务器也使用该端口。

tcpcopy 0.6

nethogs 用来监控流量



tcpcopy安装：

tar -zxvf tcpcopy-0.6.0 .tar.gz

cd tcpcopy-0.6.0

./configure

make

make install

第一步：设置静态IP

设置静态IP是为了以后测试方便，可以略过

/etc/netword/interfaces中加入



auto eth0 \#网卡

iface eth0 inet static

address 192.168.0.94 \#IP地址

gateway 192.168.0.254 \#网关

netmask 255.255.255.0 \#子网掩码

/etc/NetworkManager/NetworkManager.conf 中设置



\[ifupdown\]

managed=true

第二步：启动在线服务器和测试服务器上的测试程序

第三步：启动测试服务器上的intercept

\#modprobe ip\_queue

\#iptables -I OUTPUT -p tcp --sport &lt;port&gt; -j QUEUE 

\#intercept

第四步：启动在线服务器上的tcpcopy

\#tcpcopy  -x &lt;port&gt;-192.168.0.96:&lt;port&gt; -n 100

第五步：通过nethogs查看压力是否上来

nethogs安装：



\#apt-get install nethogs

nethogs使用：



\#nethogs eth0

