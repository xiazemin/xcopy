# libpcap

libpcap是unix/linux平台下的网络数据包捕获函数包，大多数网络监控软件都以它为基础。Libpcap可以在绝大多数类unix平台下工作.

Libpcap应用程序框架

Libpcap提供了系统独立的用户级别网络数据包捕获接口，并充分考虑到应用程序的可移植性。Libpcap可以在绝大多数类unix平台下工作，参考资料 A 中是对基于 libpcap 的网络应用程序的一个详细列表。在windows平台下，一个与libpcap 很类似的函数包 winpcap 提供捕获功能，其官方网站是http://winpcap.polito.it/。

Libpcap 软件包可从 http://www.tcpdump.org/ 下载，然后依此执行下列三条命令即可安装，但如果希望libpcap能在linux上正常工作，则必须使内核支持"packet"协议，也即在编译内核时打开配置选项 CONFIG\_PACKET\(选项缺省为打开\)。

./configure

make

make install

libpcap源代码由20多个C文件构成，但在 Linux系统下并不是所有文件都用到。可以通过查看命令make的输出了解实际所用的文件。本文所针对的libpcap版本号为0.8.3，网络类型为常规以太网。Libpcap应用程序从形式上看很简单.

Winpcap是libpcap的Windows版本。

