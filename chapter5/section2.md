# section2

除了使用修饰符和ID组成的表达式单元，还有关键字表达式单元：gateway，broadcast，less，greater以及算术表达式。



表达式单元之间可以使用操作符" and / && / or / \|\| / not / ! "进行连接，从而组成复杂的条件表达式。如"host foo and not port ftp and not port ftp-data"，这表示筛选的数据包要满足"主机为foo且端口不是ftp\(端口21\)和ftp-data\(端口20\)的包"，常用端口和名字的对应关系可在linux系统中的/etc/service文件中找到。



另外，同样的修饰符可省略，如"tcp dst port ftp or ftp-data or domain"与"tcp dst port ftp or tcp dst port ftp-data or tcp dst port domain"意义相同，都表示包的协议为tcp且目的端口为ftp或ftp-data或domain\(端口53\)。



使用括号"\(\)"可以改变表达式的优先级，但需要注意的是括号会被shell解释，所以应该使用反斜线"\"转义为"\\(\\)"，在需要的时候，还需要包围在引号中。





1.3 tcpdump示例

注意，tcpdump只能抓取流经本机的数据包。



\(1\).默认启动



tcpdump

默认情况下，直接启动tcpdump将监视第一个网络接口\(非lo口\)上所有流通的数据包。这样抓取的结果会非常多，滚动非常快。



\(2\).监视指定网络接口的数据包



tcpdump -i eth1

如果不指定网卡，默认tcpdump只会监视第一个网络接口，如eth0。



\(3\).监视指定主机的数据包，例如所有进入或离开longshuai的数据包



tcpdump host longshuai

\(4\).打印helios&lt;--&gt;hot或helios&lt;--&gt;ace之间通信的数据包



tcpdump host helios and \\( hot or ace \\)

\(5\).打印ace与任何其他主机之间通信的IP数据包,但不包括与helios之间的数据包



tcpdump ip host ace and not helios

\(6\).截获主机hostname发送的所有数据



tcpdump src host hostname

\(7\).监视所有发送到主机hostname的数据包



tcpdump dst host hostname

\(8\).监视指定主机和端口的数据包



tcpdump tcp port 22 and host hostname

\(9\).对本机的udp 123端口进行监视\(123为ntp的服务端口\)



tcpdump udp port 123

\(10\).监视指定网络的数据包，如本机与192.168网段通信的数据包，"-c 10"表示只抓取10个包



tcpdump -c 10 net 192.168

\(11\).打印所有通过网关snup的ftp数据包\(注意,表达式被单引号括起来了,这可以防止shell对其中的括号进行错误解析\)



shell&gt; tcpdump 'gateway snup and \(port ftp or ftp-data\)'

\(12\).抓取ping包



\[root@server2 ~\]\# tcpdump -c 5 -nn -i eth0 icmp 



tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth0, link-type EN10MB \(Ethernet\), capture size 65535 bytes

12:11:23.273638 IP 192.168.100.70 &gt; 192.168.100.62: ICMP echo request, id 16422, seq 10, length 64

12:11:23.273666 IP 192.168.100.62 &gt; 192.168.100.70: ICMP echo reply, id 16422, seq 10, length 64

12:11:24.356915 IP 192.168.100.70 &gt; 192.168.100.62: ICMP echo request, id 16422, seq 11, length 64

12:11:24.356936 IP 192.168.100.62 &gt; 192.168.100.70: ICMP echo reply, id 16422, seq 11, length 64

12:11:25.440887 IP 192.168.100.70 &gt; 192.168.100.62: ICMP echo request, id 16422, seq 12, length 64

5 packets captured

6 packets received by filter

0 packets dropped by kernel

如果明确要抓取主机为192.168.100.70对本机的ping，则使用and操作符。



\[root@server2 ~\]\# tcpdump -c 5 -nn -i eth0 icmp and src 192.168.100.62



tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth0, link-type EN10MB \(Ethernet\), capture size 65535 bytes

12:09:29.957132 IP 192.168.100.70 &gt; 192.168.100.62: ICMP echo request, id 16166, seq 1, length 64

12:09:31.041035 IP 192.168.100.70 &gt; 192.168.100.62: ICMP echo request, id 16166, seq 2, length 64

12:09:32.124562 IP 192.168.100.70 &gt; 192.168.100.62: ICMP echo request, id 16166, seq 3, length 64

12:09:33.208514 IP 192.168.100.70 &gt; 192.168.100.62: ICMP echo request, id 16166, seq 4, length 64

12:09:34.292222 IP 192.168.100.70 &gt; 192.168.100.62: ICMP echo request, id 16166, seq 5, length 64

5 packets captured

5 packets received by filter

0 packets dropped by kernel

注意不能直接写icmp src 192.168.100.70，因为icmp协议不支持直接应用host这个type。



\(13\).抓取到本机22端口包



\[root@server2 ~\]\# tcpdump -c 10 -nn -i eth0 tcp dst port 22  



tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth0, link-type EN10MB \(Ethernet\), capture size 65535 bytes

12:06:57.574293 IP 192.168.100.1.5788 &gt; 192.168.100.62.22: Flags \[.\], ack 535528834, win 2053, length 0

12:06:57.629125 IP 192.168.100.1.5788 &gt; 192.168.100.62.22: Flags \[.\], ack 193, win 2052, length 0

12:06:57.684688 IP 192.168.100.1.5788 &gt; 192.168.100.62.22: Flags \[.\], ack 385, win 2051, length 0

12:06:57.738977 IP 192.168.100.1.5788 &gt; 192.168.100.62.22: Flags \[.\], ack 577, win 2050, length 0

12:06:57.794305 IP 192.168.100.1.5788 &gt; 192.168.100.62.22: Flags \[.\], ack 769, win 2050, length 0

12:06:57.848720 IP 192.168.100.1.5788 &gt; 192.168.100.62.22: Flags \[.\], ack 961, win 2049, length 0

12:06:57.904057 IP 192.168.100.1.5788 &gt; 192.168.100.62.22: Flags \[.\], ack 1153, win 2048, length 0

12:06:57.958477 IP 192.168.100.1.5788 &gt; 192.168.100.62.22: Flags \[.\], ack 1345, win 2047, length 0

12:06:58.014338 IP 192.168.100.1.5788 &gt; 192.168.100.62.22: Flags \[.\], ack 1537, win 2053, length 0

12:06:58.069361 IP 192.168.100.1.5788 &gt; 192.168.100.62.22: Flags \[.\], ack 1729, win 2052, length 0

10 packets captured

10 packets received by filter

0 packets dropped by kernel

\(14\).解析包数据



\[root@server2 ~\]\# tcpdump -c 2 -q -XX -vvv -nn -i eth0 tcp dst port 22

tcpdump: listening on eth0, link-type EN10MB \(Ethernet\), capture size 65535 bytes

12:15:54.788812 IP \(tos 0x0, ttl 64, id 19303, offset 0, flags \[DF\], proto TCP \(6\), length 40\)

    192.168.100.1.5788 &gt; 192.168.100.62.22: tcp 0

        0x0000:  000c 2908 9234 0050 56c0 0008 0800 4500  ..\)..4.PV.....E.

        0x0010:  0028 4b67 4000 4006 a5d8 c0a8 6401 c0a8  .\(Kg@.@.....d...

        0x0020:  643e 169c 0016 2426 5fd6 1fec 2b62 5010  d&gt;....$&\_...+bP.

        0x0030:  0803 7844 0000 0000 0000 0000            ..xD........

12:15:54.842641 IP \(tos 0x0, ttl 64, id 19304, offset 0, flags \[DF\], proto TCP \(6\), length 40\)

    192.168.100.1.5788 &gt; 192.168.100.62.22: tcp 0

        0x0000:  000c 2908 9234 0050 56c0 0008 0800 4500  ..\)..4.PV.....E.

        0x0010:  0028 4b68 4000 4006 a5d7 c0a8 6401 c0a8  .\(Kh@.@.....d...

        0x0020:  643e 169c 0016 2426 5fd6 1fec 2d62 5010  d&gt;....$&\_...-bP.

        0x0030:  0801 7646 0000 0000 0000 0000            ..vF........

2 packets captured

2 packets received by filter

0 packets dropped by kernel

总的来说，tcpdump对基本的数据包抓取方法还是较简单的。只要掌握有限的几个选项\(-nn -XX -vvv -i -c -q\)，再组合表达式即可。





