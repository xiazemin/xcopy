# tcpdump的底层原理

1.1.1.1  如何实现

先来看看包传递过来的流程，如下图。包从网卡到内存，到内核态，最后给用户程序使用。我们知道tcpdump程序运行在用户态，那如何实现从内核态的抓包呢？

![](/assets/libpcab.png)

这个就是通过libpcap库来实现的，tcpdump调用libpcap的api函数，由libpcap进入到内核态到链路层来抓包,如下图。图中的BPF是过滤器，可以根据用户设置用于数据包过滤减少应用程序的数据包的包数和字节数从而提高性能。BufferQ是缓存供应用程序读取的数据包。我们可以说tcpdump底层原理其实就是libpcap的实现原理。

![](/assets/libpcab2.png)

而libpcap在linux系统链路层中抓包是通过PF\_PACKET套接字来实现的\(不同的系统其实现机制是由差异的\)，该方法在创建的时候，可以指定第二参数为SOCK\_DGRAM或者SOCK\_RAW，影响是否扣除链路层的首部。

```
        libpcap在内核收发包的接口处将skb\_clone\(\)拿走的包.
```

1.1.1.2  libpcap

当在系统中输入tcpdump –version的时候，输出的其实还有libpcap，足见其在tcpdump中的地位。



            其实最早的编译系统和过滤引擎是在tcpdump项目中的，后来为了编译其他抓包的应用，将其独立出来。现在libpcap提供独立于平台的库和API，来满足执行网络嗅探。



tcpdump.c正式使用libpcap里的函数完成两个最关键的动作：获取捕获报文的接口，和捕获报文并将报文交给callback。



libpcap支持“伯克利包过滤（BPF）”语法。BPF能够通过比较第2、3、4层协议中各个数据字段值的方法对流量进行过滤。Libpcap的使用逻辑如下图：

![](/assets/libpcap4.png)





       如果愿意，大家也可以基于libpcap开发一个类似tcpdump的抓包工具。需要注意的是如果使用分组捕获设备，只能在单个接口上接收到达的分组。

