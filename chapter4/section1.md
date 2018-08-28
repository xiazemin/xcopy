# tcpcopy的工作原理

TCPCopy 是一种请求复制（复制基于 TCP 的 packets）工具 ，通过复制在线数据包，修改 TCP/IP 头部信息，发送给测试服务器，达到欺骗测试服务器的TCP 程序的目的，从而为欺骗上层应用打下坚实基础。

tcpcopy 不是基于应用层的复制，而是基于底层数据包的请求复制，这么做的好处是：可以做到无需穿透整个协议栈，路程最短的，可以从数据链路层抓请求包，从数据链路层发包，路程一般的，可以在IP层抓请求包，从IP层发出去，不管怎么走，只要不走TCP，对在线的影响就会小得多。

tcpcopy 经历了三次架构调整，这三次架构的基本原理都一样，本质是利用在线数据包信息，模拟tcp客户端协议栈，欺骗测试服务器的上层应用服务。由于tcp交互是相互的，一般情况下需要知道测试服务器的响应数据包信息，才能利用在线请求数据包，构造出适合测试服务器的请求数据包，因此只要基于数据包的方式，无论怎么实现（除非是tcp协议改的面目全非），都需要返回响应包的相关信息。

三种架构的差别就在于在什么地方截获响应包。

具体这三种架构的差别请看上面文章，简单差别如下：

方案一：tcpcopy是从数据链路层\(pcap接口\)抓请求数据包，发包是从IP层发出去

方案二：tcpcopy默认从IP层抓包，从IP层发包

方案三：跟方案一一样，不过引入了独立的 intercept\(assistant server\)

tcpcopy 架构

tcpcopy运行需要intercept的支持，tcpcopy负责抓包和发包工作，而intercept负责截获应答包

它的数据流转和部署架构如下图：

![](/assets/intercept.png)

具体的生产环境和镜像环境数据传递流程图如下：

![](/assets/tcpcopyonline.png)

TCPcopy 从数据链路层 copy 端口请求，然后更改目的 ip 和目的端口。

将修改过的数据包传送给数据链路层，并且保持 tcp 连接请求。

通过数据链路层从 online server 发送到 test server。

在数据链路层解封装后到达 nginx 响应的服务端口。

等用户请求的数据返回结果后，回包走数据链路层。

通过数据链路层将返回的结果从 test server 发送到 assistant server。注：test server 只有一条默认路由指向 assistant server。

数据到达 assistant server 后被 intercept 进程截获。

过滤相关信息将请求状态发送给 online server 的 tcpcopy，关闭 tcp 连接。

对于tcpcopy，configure的时候加上--offline 

--offline replay TCP streams from the pcap file 

执行的时候，加上-i参数，用来指定pcap文件地址 

对于intercept，不用变化



