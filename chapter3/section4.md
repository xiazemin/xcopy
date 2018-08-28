# 回放

1.1  流量回放测试技术简介



1.1.1  Why do we need replay test?



Field test: 真实场景测试，优点：真实；缺点：无法测试特定场景下设备的表现以及压力测试



Lab test: 实验室环境测试，优点：见上；缺点：无法提供大量用户和应用程序、复杂网络结构的真实场景



Replay test: field test 和 lab test 的强强联合，先在 filed test 场景下将真实流量捕获，存成快照，然后将快照作为 lab test 的背景流量，在其上加入各种特定测试目标而设计的流量，对设备进行测试。



1.1.2  How to replay？



Step1:capture traffic  ----à   嗅探工具



Step2: split the network traffic （真实网络的流量一般是有进有出的）---à tcpprep



Step3: make the replay network traffic as real as it is like in the Field Test environment ---à tcprewrite



Step4: replay ---à tcpreplay



1.1.3  Replay tools



Name



Complete Connection



Stateful



Selected Replay



TCPreplay



No required



No



No



Tomahawk



Required



Yes



No



Mokey



Not required



Yes



No



Avalanche



Required



Yes



No



SocketReplay



Not required



Yes



Yes



1.1.1  Some disadvantage



1.Obviously the biggest limitation of tcpreplay is it doesn't come with a library of pcaps



 



2.In my view, the biggest limitation is that replaying captured packets an overly simplified manner of modeling real world attacks. Today's exploit code is a lot "smarter" than simple PoC that send the same fixed data on each run. modern exploits make runtime decisions based on the state of the target system/application and several other things



1.2  Tcpreplay 套件



1.2.1  套件简介



Tcpreplay是一个工具套件，用来测试各种网络设备，包括：交换机、路由器、防火墙、NIDS和IPS等。它使用嗅探工具抓取的数据包，允许你拆分客户端和服务端流量，重写2、3、4层头信息，最终将流量回放到网络中。3.4 版本包括以下部分：



 



u  Tcpprep:将流量拆分为客户端和服务端两个方向，并存放为缓存文件



u  Tcprewrite:重写pcap文件的TCP/IP层和数据链路层的头信息



u  Tcpreplay:以可控的速度将pcap文件回放到网络中



u  Tcpreplay-edit:在tcpreplay基础上增加编辑功能



u  Tcpbirdge:桥接两个不同网段



u  Tcpcapinfo:pcap 文件解码器和编译器



1.2.2  Tcpprep工具的使用



1.2.2.1  简介



有些回放测试场景下，我们希望让客户端流量从被测设备（DUT）的一端进入，服务端流量从另一端进入。Tcpprep的作用是通过对pcap文件进行一些计算，将结果存放为cache文件的形式，发送的时候加上该cache文件就可以将流量拆分为2个方向，这比边发送边计算效率要高。



1.2.2.2  流量拆分模式



8种拆分模式（具体算法 http://tcpreplay.synfin.net/wiki/tcpprep ）：



 



u  Auto/Bridge



u  Auto/Router



u  Auto/Client



u  Auto/Server



u  IPv4/v6 matching CIDR



u  IPv4/v6 matching Regex



u  TCP/UDP Port



u  MAC address



 



可以分为自动运算和匹配2大类。



 



在自动模式下，程序根据数据包的特征自动区分client 端和server 端数据。4种自动模式的基本算法是一样的，区别在于基本算法算完之后，无法划分的那些包的归属方式不同。



 



当前版本的客户端特征为：



u  Sending a TCP Syn packet to another host



u  Making a DNS request



u  Recieving an ICMP port unreachable



服务端特征为:



u  Sending a TCP Syn/Ack packet to another host



u  Sending a DNS Reply



u  Sending an ICMP port unreachable



 



 



匹配模式：给出一个IP列表或者正则表达式，或者端口，或者MAC，匹配到的认为是服务端流量



$ tcpprep --cidr=10.0.0.0/8,172.16.0.0/12 --pcap=input.pcap --cachefile=input.cache



Cidr给出的两个网络的流量被认为是服务端



$ tcpprep --regex="\(10\|20\)\..\*" --pcap=input.pcap --cachefile=input.cache



10或20开头的IP被认为是服务端



$ tcpprep --port --services=/etc/services --pcap=input.pcap --cachefile=input.cache



一般 0-1023被认为是服务端，这个范围可以自定义



$ tcpprep --mac=00:21:00:55:23:AF,00:45:90:E0:CF:A2 --pcap=input.pcap --cachefile=input.cache



匹配到MAC列表的被认为是服务端



 



1.2.2.3  Include/exclude列表



可以通过设置include列表和exclude列表，让pcap在计算的时候减少负担。



 



Include：



// 它将只拆分源地址从 10.0.0.0/8,192.168.0.0/16这两个网络出来的流量



$ tcpprep --include=S:10.0.0.0/8,192.168.0.0/16 --pcap=input.pcap --cachefile=input.cache &lt;other args&gt;



// 它将只拆分目的地址为 10.0.0.0/8,192.168.0.0/16这两个网络的流量



$ tcpprep --include=D:10.0.0.0/8,192.168.0.0/16 --pcap=input.pcap --cachefile=input.cache &lt;other args&gt;



// 它拆分源和目的地址为 10.0.0.0/8,192.168.0.0/16这两个网络的流量（即这两个网络之间的通讯流量）



$ tcpprep --include=B:10.0.0.0/8,192.168.0.0/16 --pcap=input.pcap --cachefile=input.cache &lt;other args&gt;



// 它拆分源或目的地址为 10.0.0.0/8,192.168.0.0/16这两个网络的流量（只要从两个网络中的至少一个经过的流量）



$ tcpprep --include=E:10.0.0.0/8,192.168.0.0/16 --pcap=input.pcap --cachefile=input.cache &lt;other args&gt;



 



Exclude：



// 拆分流量中除了源地址来自于10.0.0.0/8,192.168.0.0/16这两个网络的其他流量



$ tcpprep --exclude=S:10.0.0.0/8,192.168.0.0/16 --pcap=input.pcap --cachefile=input.cache &lt;other args&gt;



1.2.2.4  打印流量拆分信息



u  评论



$ tcpprep --print-comment=input.cache



u  状态



$ tcpprep --print-stats=input.cache



u  详细信息



$ tcpprep --print-info=input.cache



 



1.2.3  Tcprewrite工具的使用



1.2.3.1  简介



实现对流量包各层头信息的重写



1.2.3.2  基本用法



u  Layer2数据链路层的重写



数据链路层的特点是网络类型比较多，tcprewrite支持的类型：



n  输出 Plugins supported in output mode



Ethernet \(enet\)



Cisco HDLC \(hdlc\)



User defined Layer 2 \(user\)



n  输入Plugins supported in input mode



Ethernet



Cisco HDLC



Linux SLL



BSD Loopback



BSD Null



Raw IP



802.11



Juniper Ethernet \(version &gt;= 4.0\)



    



Tcprewrite重写第二层的举例：



n  DLT\_EN10MB \(Ethernet\)



// 下面的指令将使所有流量的源MAC变成00:44:66:FC:29:AF，目标MAC变成00:55:22:AF:C6:37



$ tcprewrite --enet-dmac=00:55:22:AF:C6:37 --enet-smac=00:44:66:FC:29:AF --infile=input.pcap --outfile=output.pcap



 



【以下这两点没有试验过，只是自己的理解】



n  DLT\_CHDLC \(Cisco HDLC\)



//Cisco HDLC has two fields in the Layer 2 header: address and control. Both can be set using this plugin: \(通过添加一些插件支持非以太网的数据链路层包重写\)



--hdlc-address



--hdlc-control



n  DLT\_USER0 \(User Defined\)



//The user defined DLT option allows you to create any DLT/Layer2 header of your choosing by using the following two options: （支持用户自定义链路层数据格式）



--user-dlt - Set pcap DLT type



--user-dlink - Set packet layer 2 header



 



u  Layer3 网络层的重写



n  使流量在两个主机之间传递



$ tcprewrite --endpoints=10.10.1.1:10.10.1.2 --cachefile=input.cache --infile=input.pcap --outfile=output.pcap --skipbroadcast



n  使流量在一个IP范围内传递



$ tcprewrite --seed=423 --infile=input.pcap --outfile=output.pcap



n  是一个网段内的流量变成另一个网段内的流量



$ tcprewrite --pnat=10.0.0.0/8:172.16.0.0/12,192.168.0.0/16:172.16.0.0/12 --infile=input.pcap --outfile=output.pcap –skipbroadcast



n  修改部分协议字段再发送流量



$ tcprewrite --tos=50 --infile=input.pcap --outfile=output.pcap



$ tcprewrite --flowlabel=67234 --infile=input.pcap --outfile=output.pcap



 



u  Layer4 TCP/UDP层的重写



n  重新映射端口



$ tcprewrite --portmap=80:8080,22:8022 --infile=input.pcap --outfile=output.pcap



n  修改部分字段



$ tcprewrite --fixcsum --infile=input.pcap --outfile=output.pcap



 



u  Layer5-7 层的重写



由于包在传输过程中是分段的，所以抓到的包里有些应用层数据已经丢失，针对这一点，tcprewrite提供了一些参数去‘修理’这个包



//Pad the packets with 0x00:



$ tcprewrite --fixlen=pad --infile=input.pcap --outfile=output.pcap



//Rewrite the packet header length fields to match the captured packet length:



$ tcprewrite --fixlen=trunc --infile=input.pcap --outfile=output.pcap



//Delete the packet from the pcap file:



$ tcprewrite --fixlen=del --infile=input.pcap --outfile=output.pcap



 



不同的网络对‘能发送的最大的数据段大小MTU’限制不同，tcprewrite 在3..3版本的时候提供了部分参数处理这个问题



 



//Truncate packets to the MTU length \(default 1500 bytes\):



$ tcprewrite --mtu-trunc --infile=input.pcap --outfile=output.pcap



//Truncate packets to a user defined MTU \(1000 bytes\):



$ tcprewrite --mtu=1000 --mtu-trunc --infile=input.pcap --outfile=output.pcap



//Use IP fragmentation to break up large packets into smaller ones. As of v3.3.0 you can use the fragroute engine to fragment IP packets into smaller pieces to fit within the MTU. The way to do this is to create a text file frag.cfg with the contents



ip\_frag 1400



and run tcprewrite like so:



$ tcprewrite --fragroute=frag.cfg --infile=input.pcap --outfile=output.pcap



 



1.2.4  Tcpreplay工具的使用



1.2.4.1  简介



Tcpreplay 到目前为止经历了3个版本的变化，1.X 实现了读包和回放功能，2.X 实现了各种重写功能，3.X 将读包（解析包）功能封装为tcpprep，将重写部分功能封装为 tcprewrite，而在tcpreplay里封装了强大的发包功能



1.2.4.2  基本用法



u  最简单的方式



\# tcpreplay --intf1=eth0 sample.pcap



 



u  控制发送速度



\# tcpreplay --topspeed --intf1=eth0 sample.pcap



\# tcpreplay --mbps=10.0 --intf1=eth0 sample.pcap



\# tcpreplay --multiplier=7.3 --intf1=eth0 sample.pcap



\# tcpreplay --multiplier=0.5 --intf1=eth0 sample.pcap



\# tcpreplay --pps=25 --intf1=eth0 sample.pcap



\# tcpreplay --oneatatime --verbose --intf1=eth0 sample.pcap



 



u  控制发送次数



\# tcpreplay --loop=10 --intf1=eth0 sample.pcap



\# tcpreplay --loop=0 --intf1=eth0 sample.pcap



 



1.2.4.3  高级



u  同时从两个端口发送数据



\# tcpreplay --cachefile=sample.prep --intf1=eth0 --intf2=eth1 sample.pcap



\# tcpreplay --dualfile --intf1=eth0 --intf2=eth1 side-a.pcap side-b.pcap



 



u  根据不同系统选择时间延迟函数



发包速度的控制，tcpreplay在实现的时候使用了延迟函数的概念，这涉及到底层OS很多特性，在不同系统上使用tcpreplay，根据系统不同选择不同的延迟函数



--timer=abstime



--timer=gtod



--timer=nano



--timer=rdtsc



--timer=ioport



--timer=select



u  一些建议



1）推荐的速度控制函数：--topspeed、--pps、--pps-multi（--pps-multi 比 –pps 每秒多发一个包，所以速度更快）； --mbps、--multiplier 在对性能要求比较高的情况下不推荐使用，因为需要大量的运算。



 



2）虚拟技术，如 vmware 会对发包的速度控制造成很大影响



1.2.5  Tcpreplay套件使用示例



http://tcpreplay.synfin.net/wiki/usage\#UsageExamples

