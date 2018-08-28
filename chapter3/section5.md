# tcpreplay工具安装-配置-实例

TCPReplay是一个非常强大的集成工具，其中主要包括TCPReplay、TCPPrepare、TCPrewrite工具，根据其具体参数对.pcap文件进行更改，然后重放数据包。



 



使用条件：Linux操作系统、双网卡



对TCPReplay工具分3个部分来讲解，第一部分安装一个操作linux操作系统、第二部分安装TCPReplay以及相关组件、举一个实例来分析相关参数意义。



一．          Ubuntu12的USB安装过程或者硬盘安装过程，在WIN7操作系统上安装；



1、 使用USB镜像安装ubuntu12操作系统的安装过程：http://www.linuxidc.com/Linux/2012-11/74695.htm；



2、 使用硬盘安装ubuntu12操作系统的安装过程：http://www.linuxidc.com/Linux/2012-11/73500.htm；



二．          TCPReplay的准备安装过程



首先推荐一个网站：http://tcpreplay.synfin.net/ ，上面有Tcpreplay的安装包和很多文档，包括手册、man页和FAQ等。



Tcpreplay是一系列工具的总称，包括tcpreplay、tcprewrite和tcpprep等工具，它可以用来在Unix系统或者linux系统上重放网络包。这些包是由tcpdump、ethereal和wireshark等软件抓取到的，即pcap格式的数据包。



安装Tcpreplay包时，默认情况下是安装了下面这些工具的, 用于准备发包的cache, 重写报文等:



（1）tcpprep: 这个工具的作用就是划分客户端和服务器，区分pcap数据包的流向，即划分那些包是client的，哪些包是server的，一会发包的时候client包从一个网卡发，另一个server的包可能从另一个网卡发。



（2）tcprewrite：这个工具的作用就是来修改报文，主要修改2层，3层，4层报文头，即MAC地址，IP地址和PORT地址。



（3）tcpreplay: 这是最终真正发包使用的工具，可以选择主网卡、从网卡、发包速度等。



Tcpreplay安装条件



Tcpreplay要实现它的功能要用到其它一些库：



1）libpcap库：由于tcpreplay在使用过程中主要依赖于libpcap库，因此在安装tcpreplay之前需要先安装libpcap，否则在安装tcpreplay的时候会提示你libpcap没有安装而安装失败。但如果希望libpcap能在linux上正常工作，则必须使内核支持"packet"协议，也即在编译内核时 打开配置选项 CONFIG\_PACKET\(选项缺省为打开\)。



Libpcap可以从以下链接下载：http://www.tcpdump.org/  可以用源码安装，比较简单。



2）tcpdump： 这个不是必须的，这个工具的主要作用就是来解码数据包，在linux系统下查看pcap文件内的数据包，也可以用它来进行抓包。在安装tcpreplay时可以选择安装它也可以不选择安装。（个人建议也一起安装，以防止在使用tcpreplay时发生其他错误。）



3）libnet库。也不是必须的一个库。Tcpreplay也可用它来发送数据包，但由于libnet自身的bug比较多且已不再有人维护，Tcpreplay未来版本有可能取消对它的支持。（目前的版本还需要这个库的支持，因此建议也安装）



另外，你所使用的linux系统需要安装了GCC编译器，否则无法安装所有工具。



Tcpreplay安装过程



（1）首先来说明一下如何安装libpcap库（在安装tcpreplay之前需要先安装libpcap），安装libpcap之前需要首先安装m4、bison，flex，否则会出现错误：



1）打开网址：www.tcpdump.org/ 下载 libpcap-1.1.1.tar.gz \(512.0KB\) 软件包，通过命令tar zxvf libpcap-1.1.1.tar.gz 解压文件，并将其放入自定义的安装目录。



2）打开网址：flex.sourceforge.net/ 下载 flex-2.5.35.tar.gz \(1.40MB\) 软件包，通过 tar zxvf flex-2.5.35.tar.gz  解压文件，并将其放入上述自定义的安装目录中。



注意：如果没有编译安装此文件，在编译安装libpcap时，就会出现 “configure: error: Your operating system\'s lex is insufficient to compile libpcap.”的错误提示。



3）打开网址：ftp.gnu.org/gnu/bison/ 下载 bison-2.4.1.tar.gz \(1.9MB\) 软件包，通过 tar zxvf bison-2.4.1.tar.gz 解压文件，并将其放入上述自定义的安装目录中。



注意：如果没有编译安装此文件，在编译安装libpcap时，就会出现 "configure: WARNING: don\'t have both flex and bison; reverting to lex/yacc checking for capable lex... insufficient" 的错误提示。



4）打开网址：ftp.gnu.org/gnu/m4/ 下载 m4-1.4.13.tar.gz \(1.2MB\)软件包，通过 tar zxvfm4-1.4.13.tar.gz 解压文件，并将其放入上述自定义的安装目录中。



注意：如果没有编译安装此文件，在编译安装bison-2.4.1时，就会出现 “configure: error: GNU M4 1.4 is required”的错误提示。



然后，依次进入m4-1.4.13，bison-2.4.1，flex-2.5.35，libpcap-1.1.1 并执行以下命令：



\# ./configure



\# make



\# make install



命令完成后，libpcap才能正常使用。



（2）安装完libpcap后就可以安装tcpreplay了，从这个链接http://tcpreplay.synfin.net/下载tcpreplay-3.4.4.tar.gz软件包，通过 tar zxvf tcpreplay-3.4.4.tar.gz  解压文件，并将其放入上述自定义的安装目录中。然后进入tcpreplay-3.4.4，并执行以下命令：



\# ./configure



\# make



\# make install



执行完后，tcpreplay 就可以使用了，可以通过命令：tcpreplay –vesion来查看它的版本信息，tcpreplay –h来查看帮助内容。 



三.Tcpreplay具体实例



通过一个经过测试的实例来介绍他们的用法。



测试拓扑图如下图所示



\"\[转载\]TCPReplay使用\"



其中TCPReplay机器的配置为：



OS： Ubuntu9.04内核版本：2.6.28



 



 



 



 



 



 



 



 



TCPReplay版本：3.3.2（不同版本命令可能稍有不同，具体请通过MAN查询）



网卡：Intel e1000e 双千兆



PCAP文件：test.tcpdump



测试第一步：预处理生成Cache,命令为



tcpprep -a client -i test.tcpdump -o test.cache



这条命令将PCAP文件分成客户端和服务端，默认为客户端。发送时packet将分别从客户端和服务端发出。



测试第二步：重写IP地址和MAC地址，命令为：



tcprewrite -e 192.85.2.2:192.85.1.2 --enet-dmac=00:15:17:2b:ca:15,00:15:17:2b:ca:14 --enet-smac=00:10:f3:19:79:87,00:10:f3:19:79:86 -c test.cache -i test.tcpdump -o 1.pcap



这条命令将eth0设为客户端接口，eth1设为服务端接口，重写了IP和MAC,可通过wireshark等工具打开1.pcap，查看修改是否成功，其中IP地址谁在前，默认谁为服务端，其中IP/MAC都要验证正确输入，否则在重放过程中会出错。



测试第三步：重放packet，首先为了获取更高的发送速度，可以把文件放到/dev/shm目录下，最高速度有1倍左右的加速。重放命令为：



tcpreplay -i eth0 -I eth1 -l 1000 -t -c /dev/shm/test.cache /dev/shm/1.pcap



命令分析：-i后面参数为主网卡/服务器网卡, -I后面参数为次网卡/客户端网卡； -l 后面的参数为播放次数，-t表示尽可能的快 –c后面紧跟着两个参数分别为.cache和.pcap。这条命令将文件以最高速率循环发送1000次。上述步骤通过测试，保证能够通过。

