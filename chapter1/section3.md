# Tcpreplay 根据 cach 文件来回放 pcap 文件中的数据

　tcpreplay小例子



\[root@datatest ~\]\#  tcpreplay --intf1=eth1 --intf2=eth0 –t --cachfile=cach\_test.cach http\_rewrite\_pcap

　　



　　--intf1=eth1 是指主接口是 eth1， 服务器－&gt; 客户端的数据包通过这个接口发送。



　　　　　　服务器和客户端的区分是从 tcpprep 的处理结果 cach\_test.cach 中得到的。

　　--intf2=eth0 是指从接口是 eth0，客户端－&gt;服务器的数据包通过这个接口发送。

　　--cachfile=cach\_test\_cach 是指 tcpreplay 用 下面的tcpprep 的处理结果--cach\_test.cach 来区分方向。

　　http\_rewrite.pcap 是指 tcpreplay 发送的是来自 http\_rewrite.pcap 这个文件中的数据包 。



 



\[root@datatest ~\]\#  tcpreplay –i eth1 –I eth0 –c ceshiftp.cach ceshiftp.pcap

 　　-i, --intf1这是数据重放的时候，的从接口。     -I, --intf2这是数据重放的时候，的主接口。　　–c是双网卡回放报文必选参数，后跟文件名



 



 



 



　　在linux下, 用ifconfig命令可以获得接口的名字。



　　但在cygwin下必须用下面的命令获得接口的名字。

　　在cygwin下获得接口的名字:



$ tcpreplay --listnics

 



　　假设在windows下的cygwin里安装的tcpreplay



复制代码

$ tcpreplay --listnics

\|Available network interfaces:

\|Alias   Name    Description

\|%0      \Device\NPF\_GenericDialupAdapter

\|        Adapter for generic dialup and VPN capture

\|%1      \Device\NPF\_{6B508B29-B3E3-4D0B-892F-02914AC9A668}

\|    Intel\(R\) 82566DM Gigabit Network Connection \(Microsoft's Packet

\|    Scheduler\)

\|%2      \Device\NPF\_{CBCE38CA-1FAD-4AEB-89DF-FD2D8EF861FA}

\|    D-Link DFE-530TX PCI Fast Ethernet Adapter \(rev.C\)

\|    \(Microsoft's Packet Scheduler\)

\|%3      \Device\NPF\_{ABB813FE-3C51-49A3-8146-16CD2C4507C3}

\|    D-Link DFE-530TX PCI Fast Ethernet Adapter \(rev.C\)

\|    \(Microsoft's Packet Scheduler\)

复制代码

 



 



 　　从中可以看出这台机器有两块网卡, 一块叫做%1, 另一块叫做%2. 下面就可以指定网卡重发包了:

　　用tcpreplay发包:



\[root@datatest ~\]\#  tcpreplay -c mgcp.cach -i %1 -I %2 out.pcap

 　　这条命令是把out.pcap包, 按照mgcp.cach里划分的主机包和客户端包的方式, 将主机包从网卡%1发出去, 将客户端包从网卡%2发出去.





　　注:修改目的IP的MAC地址的时候，MAC地址不是目的机网卡物理地址，而是源机器所连接的交换机MAC地址。



 





