# 传输介质上的嗅探

在网络的传输介质上实现嗅探是最不容易被发现和防范的一种被动攻击。攻击者不需要修改网络互联设备的配置，甚至不需要接入网络，只需物理上的接触即可获取介 质上传输的信息内容。尽管这种嗅探需要先进的专用设备才能实现，但实施起来的隐蔽性使其成为不容忽视的网络安全威胁之一。 



无线网络直接利用电磁波传输信息的特性使其不可避免的面临侦听的威胁；铜缆和非屏蔽双绞线会辐射出电磁波而导致信息泄漏；甚至是被普遍认为无法窃听的光纤，如果能够在不损坏内部结构的前提下剥去其涂敷层（现在还做不到），再使其弯曲引发辐射，照样能够达到嗅探的目的。



1.1.2  各种嗅探工具



谷歌以下关键字：\*sniff 、tcpdump 、Ethereal\(wireshark\)



1.2  Tcpdump



1.2.1  基本命令行参数



Tcpdump是linux系统下一款非常强大的嗅探器，它采用命令行方式工作，它的命令格式为：







比较常用的参数/值



-i any : 监听所有网络端口

-n : 不把网络地址显示为域名

-nn : 不显示域名和端口名

-X : 在输出行同时打印ASCII字符和HEX十六进制显示的包信息

-XX : 在-X基础上，增加了数据链路层的头部信息的显示

-v, -vv, -vvv : 逐步提高抓取信息的详细程度

-c : 抓取c个包即停止

-S : 打印绝对序列号

-e : 在输出行打印出数据链路层的头部信息

-s : 设置默认抓取的包数据的长度（如果不设置，默认只抓一个包前面 96 字节）

-w:直接将包写入文件中，并不分析和打印出来

 



几个简单的例子



 



1）.Basic communication // 打印最简单的包信息



\# tcpdump -nS



2） Basic communication \(very verbose\) // 打印比较详细的包信息



\# tcpdump -nnvvS



3） A deeper look at the traffic // 增加HEX码的打印



\# tcpdump -nnvvXS



4） Heavy packet viewing //增加抓取的字节长度



\# tcpdump -nnvvXSs 1514



 



1.1.1  过滤表达式



表达式是一个正则表达式，tcpdump利用它作为过滤报文的条件，如果一个报文满足表达式的条件，则这个报文将会被捕获。如果没有给出任何条件，则网络上所有的信息包将会被截获。



表达式的几种关键字：



 



第一种是关于类型的关键字，主要包括host，net，port, 例如 host  210.27.48.2，指明 210.27.48.2是一台主机，net  202.0.0.0 指明 202.0.0.0是一个网络地址，port 23 指明端口号是23。如果没有指定类型，缺省的类型是host.

　　　第二种是确定传输方向的关键字，主要包括src , dst ,dst or src, dst and src ,这些关键字指明了传输的方向。举例说明，src 210.27.48.2 ,指明ip包中源地址是210.27.48.2 , dst net 202.0.0.0 指明目的网络地址是202.0.0.0 。如果没有指明方向关键字，则缺省是src or dst关键字。

　　　第三种是协议的关键字，主要包括fddi,ip ,arp,rarp,tcp,udp等类型。Fddi指明是在FDDI\(分布式光纤数据接口网络\)上的特定的网络协议，实际上它是"ether"的别 名，fddi和ether具有类似的源地址和目的地址，所以可以将fddi协议包当作ether的包进行处理和分析。其他的几个关键字就是指明了监听的包 的协议内容。如果没有指定任何协议，则tcpdump将会监听所有协议的信息包。



除了这三种类型的关键字之外，其他重要的关键字如下：gateway,broadcast,less,greater,还有三种逻辑运算，取非运算是 'not ' '! ', 与运算是'and','&&';或运算 是'or' ,'\|\|'；



 



 



几个简单的例子：



host // 根据IP抓包

\# tcpdump host 1.2.3.4



src, dst // 根据IP只抓一个方向的包

\# tcpdump src 2.3.4.5 

\# tcpdump dst 3.4.5.6



net // 抓取一个局域网内的包

\# tcpdump net 1.2.3.0/24



proto // 根据协议抓包

\# tcpdump icmp



port // 根据端口抓包

\# tcpdump port 3389



src, dst port // 根据端口只抓一个流向的包

\# tcpdump src port 1025 

\# tcpdump dst port 389



Port Ranges // 根据端口范围抓包

\# tcpdump portrange 21-23



Packet Size Filter // 根据包大小抓包

\# tcpdump less 32 

\# tcpdump greater 128



// 也可以下面这种形式 

\# tcpdump &gt; 32 

\# tcpdump &lt;= 128



 



1.1.2  高级应用



关键字可以组合起来构成强大的组合条件来满足人们的需要，下面举几个例子来说明



 



// 抓取源地址 10.5.2.3 目的端口 3389 的包，打印中等详细的信息，不显示域名和端口名



\#tcpdump -nnvvS  and src 10.5.2.3 and dst port 3389



 



// 抓从网络 192.168 发向网络 10 或 172.16 的流量，同时打印ASCII和HEX的信息



\#tcpdump -nvX  src net 192.168.0.0/16 and dst net 10.0.0.0/8 or 172.16.0.0/16



 



\# 抓目的地址为 192.168.0.2 的非 icmp 协议的包，每个包抓 1514 字节



tcpdump -nvvXSs 1514 dst 192.168.0.2 and src net and not icmp



 



还可以类似编程的风格，用偏移量等精确到TCP包的某些字节，如



//Show me all URGENT \(URG\) packets...



\# tcpdump 'tcp\[13\] & 32!=0'



//Show me all ACKNOWLEDGE \(ACK\) packets...



\# tcpdump 'tcp\[13\] & 16!=0'



//Show me all PUSH \(PSH\) packets...



\# tcpdump 'tcp\[13\] & 8!=0'



//Show me all RESET \(RST\) packets...



\# tcpdump 'tcp\[13\] & 4!=0'



//Show me all SYNCHRONIZE \(SYN\) packets...



\# tcpdump 'tcp\[13\] & 2!=0'



//Show me all FINISH \(FIN\) packets...



\# tcpdump 'tcp\[13\] & 1!=0'

