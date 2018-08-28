# TCPCopy 线上流量复制工具

TCPCopy是一种重放TCP流的工具，使用真实环境来测试互联网服务器上的应用程序。



一、描述：



虽然真实的实时流量对于Internet服务器应用程序的测试很重要，但是由于生产环境中的情况很负责，测试环境很难完全模拟线上环境。为了能够更真实的测试，我们开发了一款线上流量复制工具-TCPCopy，它可以使用线上真实的流量来对测试环境中的服务器进行测试。目前，TcpCopy技术已经在中国很多公司大量使用。



二、使用场景：



1）分布式压力测试



使用tcpcopy复制真实的数据来进行服务器的压力测试。有些bug只有在高并发的情况下才能够被发现。



2）仿真实验：



被证明是稳定的新系统，其bug只能在真正使用的时候才能被发现



3）回归测试



4）性能对比



三、框架：







        如Figure1中所示，tcpcopy包括两部分：tcpcopy\(client\)和intercept\(server\)（后文中统一将tcpcopy-client称为tcpcopy，将tcpcopy-server称为intercept），当tcpcopy运行在生产服务器并从生产服务器抓取请求时，inteceptor运行在辅助服务器上进行一些辅助工作，例如，响应tcpcopy。切记，测试应用程序应该运行在测试服务器上。



tcpcopy默认情况下使用socket输入技术在网络层抓取线上的数据包，做一些基本处理（包括：模拟TCP交互，网络控制，以及模拟传输层和应用层），使用socket输出技术发送数据包到测试服务器（如粉色箭头所示）



tcpcopy的测试服务器需要做的唯一操作是：设置适当的参数使响应信息发送到辅助服务器中（装intercept的服务器）（如绿色箭头所示）



intercept（默认）将响应信息传送给tcpcopy。通过抓取响应包，intercept提取响应头信息，并使用一个特殊的通道将响应头信息发送给tcpcopy（如紫色箭头所示）。当tcpcopy接受到响应头信息，它利用头信息修改在线打包器的属性并继续发送另一个包。应当注意，来自测试服务器的响应被路由到应该充当黑洞的辅助服务器。



四、快速开始



1、获取intercept的两种方式：



    1）Download the latest intercept release.



    2）clone git://github.com/session-replay-tools/intercept.git



    2、获取tcpcopy的两种方式



    1）Download the latest tcpcopy release.



    2）clone git://github.com/session-replay-tools/tcpcopy.git



五、获取安装在辅助服务器上的intercept



    1）cd intercept

    2）./configure

    3）选择适当的配置参数

    4）make

    5）make install



六、intercept的配置参数



    --single            intercept运行在单机情况下

    --with-pfring=PATH  将路径设置为PF\_RING库源

    --with-debug        以debug模式编译intercept\(保存在日志文件中\)



七、获取安装在生产服务器上的tcpcopy



    1）cd tcpcopy

    2）./configure

    3）选择适当的配置参数

    4）make

    5）make install



八、tcpcopy的配置参数



    --offline 从pcap文件重放TCP流

    --pcap-capture 在数据链路层抓包（默认在网络层）

    --pcap-send 在数据链路层发包（默认在网络层）

    --with-pfring=PATH 将路径设置为PF\_RING库源

    --set-protocol-module=PATH 设置tcpcopy为外部协议模块工作

    --single 如果intercept和tcpcopy都设置为单机模式，只有一个tcpcopy和一个intercept一起工作，将会获得更好的性能

    --with-debug 以debug模式编译tcpcopy\(保存在日志文件中\)



九、运行tcpcopy



    确保tcpcopy和intercept都配置为“./configure”



    1）在运行应用程序的测试服务器上，正确设置路由命令以将响应数据包发送到辅助服务器上



    例如：



    假设61.135.233.161是辅助服务器的IP地址。 我们设置以下route命令将所有对62.135.200.x的的响应路由到辅助服务器。



    route add -net 62.135.200.0 netmask 255.255.255.0 gw 61.135.233.161



    2）在运行intercept的辅助服务器上（需要root权限或者能使用socket通信的权限）



    ./intercept -F &lt;filter&gt; -i &lt;device,&gt;



    请注意，过滤器格式与pcap过滤器相同。

    例如：./intercept -i eth0 -F 'tcp and src port 8080' -d



    intercept将捕获基于TCP应用的响应，该应用监听在设备的8080端口上



    3）生产服务器中（需要root权限或者能使用socket通信的权限）



    ./tcpcopy -x localServerPort-targetServerIP:targetServerPort -s &lt;intercept server,&gt; 



    \[-c &lt;ip range,&gt;\]



    例如（假设61.135.233.160是目标服务器的IP地址）：



    ./tcpcopy -x 80-61.135.233.160:8080 -s 61.135.233.161 -c 62.135.200.x



    tcpcopy将抓取当前服务器上80端口的数据包，修改客户端IP地址为62.135.200.x，将这些数据包发送到ip地址为61.135.233.160，端口为8080的测试服务器，并且连接61.135.233.161，告诉intercept将响应数据包发送给它（tcpcopy）

    虽然“-c”参数是可选的，但在此设置以便简化路由命令。



十、注意



    1）只能在linux上测试\(kernal 2.6 or above\)

    2）tcpcopy可能丢包，因此丢失请求

    3）root权限或socket权限是必须的\(例如 setcap CAP\_NET\_RAW = ep tcpcopy\)

    4）TCPCopy现在只支持客户端启动的连接

    5）TCPCopy不支持使用SSL / TLS的服务器应用程序的重放

    6）对于MySQL会话重放，请参考 https://github.com/session-replay-tools

    7）不应该在辅助服务器上设置ip转发

    8）请执行“./tcpcopy -h”或“./intercept -h”以获取更多详细信息



十一、影响因素



    有几个因素可能影响TCPCopy，将在以下部分中详细介绍：



    1）抓包接口



    tcpcopy默认使用套接字输入接口在网络层抓取生产服务器的数据包。在系统忙时，系统内核可能会丢包。



    如果你配置tcpcopy的参数“--pcap-capture”，tcpcopy将在数据链路层抓包，也可以过滤内核中的数据包。在PF\_RING资源中，当使用pcap捕获时，tcpcopy将丢失更少的数据包。



    或许抓请求包的最好方式是通过交换机镜像入口的数据包，然后通过负载均衡器将巨大的流量划分到几台机器



    2）发送接口



    tcpcopy默认使用套接字输出接口在网络层发送数据包到测试服务器。如果你想避免IP连接跟踪问题或者获得更好的性能表现，配置tcpcopy的参数“--pcap-send”，设置适当的参数，tcpcopy可以在数据链路层发送数据包到测试服务器。



    3）数据包在通往测试服务器的路上



    当一个数据包被tcpcopy发送时，它可能在到达测试服务器前遭到很多挑战。由于数据包中的源IP地址依然是终端用户的IP地址（默认情况下）而不是生产服务器的IP地址，一些安全设备可能将该包削弱或当做伪造的包丢弃它。这种情况下，你在测试服务器使用tcp抓包工具，可能抓取不到期望的终端用户的数据包。要确定你是否正处于这种情况下，你可以使用同一网段下的测试服务器做个小测试。如果数据包能被成功的发送到同一网段的测试服务器，而不能发送到不同网段的测试服务器，那么证明你的数据包在半路被丢弃了。



    为了解决这个问题，我们建议将tcpcopy、测试服务器、intercept部署在同一个网段内。在同一网段中有一个代理的帮助下还有另一个解决方案，tcpcopy可以向代理发送数据包，然后代理会将相应的请求发送到另一个网段中的测试服务器。



    注意，在同一网段中的一个虚拟机上部署目标服务器应用程序可能面临上述问题



    4）测试服务器的路由



    测试服务器可能设置了反向过滤技术，可以检查包中源IP地址是否是被伪造的。如果是，则该包在网络层被丢弃。



    如果在测试服务器中能用tcp抓包工具抓到包，但是测试服务器上的应用程序接收不到任何请求，你应该检查你是否有类似反向过滤技术的设置。如果设置了，你不得不移除相关的设置来让数据包通过网络层。



    也有些其他原因可能导致tcpcopy不能正常工作，例如防火墙设置问题。



    5）测试服务器上的应用程序



    测试服务器上的应用程序可能不能及时处理所有的请求。一方面，应用中的bug导致请求很长时间得不到响应；另一方面，一些TCP层以上的协议只处理socket缓冲中的第一个请求，将剩下的请求留在socket缓冲中不处理。



    6）辅助服务器的路由



    你不应该设置ip转发为true或者辅助服务器不能作为一个黑洞工作。

