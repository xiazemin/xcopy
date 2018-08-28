# Golang 基于libpcap/winpcap的底层网络编程——gopacket安装

Go简介

Go是一种编译型语言，它结合了解释型语言的游刃有余，动态类型语言的开发效率，以及静态类型的安全性。



语法类似C/C++，但是又带有一点python的味道



其中个人认为最出色的特点就是他的包管理，你可以在GoDoc、Go Search上找到非常多有用的包以及文档



同时只需执行 go get path \( e.g. go get https://godoc.org/github.com/google/gopacket\)就可以将包下载到本地



gopacket简介

gopacket是google实现的一个基于libpcap的包，可以在GoDoc上找到该包的相关文档



它包含许多子包，提供了丰富的函数库，通过导入gopacket就可以使用libpcap提供的大多数API



gopacket安装

虽然gopacket是基于libpcap实现的，但是安装winpcap同样适用



不过不管选择libpcap还是winpcap，在使用都需要先进行相应的安装到本地（笔者安装的是winpcap）



首先，通过go get https://godoc.org/github.com/google/gopacket，该命令会将其下载到GOPATH \(e.g. E:/GoWorkspace\)



但在正式使用它进行开发之前还有几个注意事项：



编写代码test.go



//测试gopacket是否能够正常使用

package main



import \(

    "fmt"

    \_ "github.com/google/gopacket"

    \_ "github.com/google/gopacket/pcap"

\)



func main\(\) {

    fmt.Println\("It works!"\)

}

运行go run test.go，若成功输出，那么请跳过后文，开始你的gopacket之旅吧！



但可能会出现如下错误



\# github.com/google/gopacket/pcap

E:\GoWorkspace\src\github.com\google\gopacket\pcap\pcap.go:21:10: fatal error: pcap.h: No such file or directory

 \#include &lt;pcap.h&gt;

          ^~~~~~~~

compilation terminated.

笔者也遇到了这个错误，google了很久并没有找到任何有用的解决方案，之后看了gopacket源码才解决



究其原因就是在于github.com/google/gopacket/pcap/pcap.go这个文件



//截取pcap.go的11-21行如下

\#cgo solaris LDFLAGS: -L /opt/local/lib -lpcap

\#cgo linux LDFLAGS: -lpcap

\#cgo dragonfly LDFLAGS: -lpcap

\#cgo freebsd LDFLAGS: -lpcap

\#cgo openbsd LDFLAGS: -lpcap

\#cgo darwin LDFLAGS: -lpcap

\#cgo windows CFLAGS: -I C:/WpdPack/Include        //问题出在 17-19行

\#cgo windows,386 LDFLAGS: -L C:/WpdPack/Lib -lwpcap

\#cgo windows,amd64 LDFLAGS: -L C:/WpdPack/Lib/x64 -lwpcap

\#include &lt;stdlib.h&gt;

\#include &lt;pcap.h&gt;

由于pcap.go中写了一个C的wrapper，但是写死了libpcap/winpcap的安装路径 pcap.go:L17-19 ，所以如果安装位置并非默认则会报上述错误



解决方案：



卸载并重新安装libpcap/winpcap到默认位置

修改github.com/google/gopacket/pcap/pcap.go文件

前者太过繁琐，笔者选择后者，将pcap.go 17-19行的三个路径改为对应的安装路径即可

