# section4

1）发起请求的客户端所在机器，不能同时运行相应的intercept，因为响应数据包路由回来后，这台机器的tcp层会发送reset数据包给测试服务器，这样就会干扰测试的进行。

2）在线服务和测试服务不要在一台机器

      如果在线服务响应的目的ip地址和测试服务响应的目的ip地址是一样的，路由设置的时候，是无法区分在线的响应和测试的响应

3）对于外网应用，由于客户端ip地址来自于世界各地，路由策略如下：



  a）用两个网卡，一个外网网卡，一个内网网卡，让外网请求都路由到第二台测试服务器上面去



   比如改变测试服务器上面的默认路由：

   route del default gw 真正的网关ip地址

   route add default gw 辅助服务器的ip地址



   



 b）利用tcpcopy的-c参数，修改客户端源ip地址，这样就方便设置路由



    比如：./tcpcopy -x 11311-10.100.10.31:11511 -s 10.100.10.32 -c 192.168.100.x



     相应路由设置：



    route add -net 192.168.100.0 netmask 255.255.255.0 gw 10.100.10.32







4）如果是在同一网段利用外网地址访问，在机器B上面设置去往机器A的响应，走机器C，那么设置默认外网网卡路由不会生效，需要显式指定，比如：



   route add -host 机器A的外网ip地址  gw 机器C的外网ip地址



5）如果是内网应用，由于客户端ip地址少，建议采用如下：

route add -host 内网客户端ip地址 gw 辅助服务器的ip地址

或者

//如果客户端ip地址来自于其它网段的话

route add -net xxx.xxx.xxx.0 netmask 255.255.255.0 gw 辅助服务器的ip地址



不要采用默认网关的方式





6）如果tcpcopy遇到大量“unsend:too many packets”的报警，请采用raw socket方式来抓请求数据包



7）如果客户端来自于同一网段，那么响应包可能会直接通过mac地址返回给客户端，导致路由设置不起作用，响应包不会被intercept所截获，导致复制失败

    解决策略有两个：



   1）检测路由命令是否有冲突，导致响应包直接返回给客户端



   2）tcpcopy运行的时候通过-c参数来改变客户端的ip地址为不同网段的ip地址，就可以解决此问题。





8）如果同时有内网访问和外网访问，应该分别针对外网应用和内网应用，设置相应路由



9）运行intercept的辅助服务器，为方便路由设置，最好要和测试服务器在同一个网段，而且不要设置ip\_forward





