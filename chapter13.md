# 解码抓取的数据

我们可以使用原始数据包，并且可将其转换为已知格式。它与不同的层兼容，所以我们可以轻松访问以太网，IP和TCP层.layers包是进入库中新增的，在底层PCAP库中不可用。这是一个令人难以置信的有用的包，它是gopacket库的一部分。它允许我们容易地识别包是否包含特定类型的层。该代码示例将显示如何使用层包来查看数据包是以太网，IP和TCP，并轻松访问这些头文件中的元素。 

查询有效载荷取决于所涉及的所有层。每个协议是不同的，必须相应地计算。这就是layer包的魅力所在。gopacket的作者花了时间为诸如以太网，IP，UDP和TCP等众多已知层创建了相应类型。有效载荷是应用层的一部分。



package main



import \(

    "fmt"

    "github.com/google/gopacket"

    "github.com/google/gopacket/layers"

    "github.com/google/gopacket/pcap"

    "log"

    "strings"

    "time"

\)



var \(

    device      string = "eth0"

    snapshotLen int32  = 1024

    promiscuous bool   = false

    err         error

    timeout     time.Duration = 30 \* time.Second

    handle      \*pcap.Handle

\)



func main\(\) {

    // Open device

    handle, err = pcap.OpenLive\(device, snapshotLen, promiscuous, timeout\)

    if err != nil {log.Fatal\(err\) }

    defer handle.Close\(\)



    packetSource := gopacket.NewPacketSource\(handle, handle.LinkType\(\)\)

    for packet := range packetSource.Packets\(\) {

        printPacketInfo\(packet\)

    }

}



func printPacketInfo\(packet gopacket.Packet\) {

    // Let's see if the packet is an ethernet packet

    ethernetLayer := packet.Layer\(layers.LayerTypeEthernet\)

    if ethernetLayer != nil {

        fmt.Println\("Ethernet layer detected."\)

        ethernetPacket, \_ := ethernetLayer.\(\*layers.Ethernet\)

        fmt.Println\("Source MAC: ", ethernetPacket.SrcMAC\)

        fmt.Println\("Destination MAC: ", ethernetPacket.DstMAC\)

        // Ethernet type is typically IPv4 but could be ARP or other

        fmt.Println\("Ethernet type: ", ethernetPacket.EthernetType\)

        fmt.Println\(\)

    }



    // Let's see if the packet is IP \(even though the ether type told us\)

    ipLayer := packet.Layer\(layers.LayerTypeIPv4\)

    if ipLayer != nil {

        fmt.Println\("IPv4 layer detected."\)

        ip, \_ := ipLayer.\(\*layers.IPv4\)



        // IP layer variables:

        // Version \(Either 4 or 6\)

        // IHL \(IP Header Length in 32-bit words\)

        // TOS, Length, Id, Flags, FragOffset, TTL, Protocol \(TCP?\),

        // Checksum, SrcIP, DstIP

        fmt.Printf\("From %s to %s\n", ip.SrcIP, ip.DstIP\)

        fmt.Println\("Protocol: ", ip.Protocol\)

        fmt.Println\(\)

    }



    // Let's see if the packet is TCP

    tcpLayer := packet.Layer\(layers.LayerTypeTCP\)

    if tcpLayer != nil {

        fmt.Println\("TCP layer detected."\)

        tcp, \_ := tcpLayer.\(\*layers.TCP\)



        // TCP layer variables:

        // SrcPort, DstPort, Seq, Ack, DataOffset, Window, Checksum, Urgent

        // Bool flags: FIN, SYN, RST, PSH, ACK, URG, ECE, CWR, NS

        fmt.Printf\("From port %d to %d\n", tcp.SrcPort, tcp.DstPort\)

        fmt.Println\("Sequence number: ", tcp.Seq\)

        fmt.Println\(\)

    }



    // Iterate over all layers, printing out each layer type

    fmt.Println\("All packet layers:"\)

    for \_, layer := range packet.Layers\(\) {

        fmt.Println\("- ", layer.LayerType\(\)\)

    }



    // When iterating through packet.Layers\(\) above,

    // if it lists Payload layer then that is the same as

    // this applicationLayer. applicationLayer contains the payload

    applicationLayer := packet.ApplicationLayer\(\)

    if applicationLayer != nil {

        fmt.Println\("Application layer/Payload found."\)

        fmt.Printf\("%s\n", applicationLayer.Payload\(\)\)



        // Search for a string inside the payload

        if strings.Contains\(string\(applicationLayer.Payload\(\)\), "HTTP"\) {

            fmt.Println\("HTTP found!"\)

        }

    }



    // Check for errors

    if err := packet.ErrorLayer\(\); err != nil {

        fmt.Println\("Error decoding some part of the packet:", err\)

    }

}

构造发送数据包

这个例子做了几件事情。首先将显示如何使用网络设备发送原始字节。这样就可以像串行连接一样使用它来发送数据。这对于真正的低层数据传输非常有用，但如果您想与应用程序进行交互，您应该构建可以识别该数据包的其他硬件和软件。接下来，它将显示如何使用以太网，IP和TCP层创建一个数据包。一切都是默认空的。要完成它，我们创建另一个数据包，但实际上填写了以太网层的一些MAC地址，IPv4的的一些IP地址和TCP层的端口号。你应该看到如何伪装数据包和仿冒网络设备.TCP层结构体具有可读取和可设置的SYN，FIN，ACK标志。这有助于操纵和模糊TCP三次握手，会话和端口扫描.pcap库提供了一种发送字节的简单方法，但gopacket中的图层可帮助我们为多层创建字节结构。



package main



import \(

    "github.com/google/gopacket"

    "github.com/google/gopacket/layers"

    "github.com/google/gopacket/pcap"

    "log"

    "net"

    "time"

\)



var \(

    device       string = "eth0"

    snapshot\_len int32  = 1024

    promiscuous  bool   = false

    err          error

    timeout      time.Duration = 30 \* time.Second

    handle       \*pcap.Handle

    buffer       gopacket.SerializeBuffer

    options      gopacket.SerializeOptions

\)



func main\(\) {

    // Open device

    handle, err = pcap.OpenLive\(device, snapshot\_len, promiscuous, timeout\)

    if err != nil {log.Fatal\(err\) }

    defer handle.Close\(\)



    // Send raw bytes over wire

    rawBytes := \[\]byte{10, 20, 30}

    err = handle.WritePacketData\(rawBytes\)

    if err != nil {

        log.Fatal\(err\)

    }



    // Create a properly formed packet, just with

    // empty details. Should fill out MAC addresses,

    // IP addresses, etc.

    buffer = gopacket.NewSerializeBuffer\(\)

    gopacket.SerializeLayers\(buffer, options,

        &layers.Ethernet{},

        &layers.IPv4{},

        &layers.TCP{},

        gopacket.Payload\(rawBytes\),

    \)

    outgoingPacket := buffer.Bytes\(\)

    // Send our packet

    err = handle.WritePacketData\(outgoingPacket\)

    if err != nil {

        log.Fatal\(err\)

    }



    // This time lets fill out some information

    ipLayer := &layers.IPv4{

        SrcIP: net.IP{127, 0, 0, 1},

        DstIP: net.IP{8, 8, 8, 8},

    }

    ethernetLayer := &layers.Ethernet{

        SrcMAC: net.HardwareAddr{0xFF, 0xAA, 0xFA, 0xAA, 0xFF, 0xAA},

        DstMAC: net.HardwareAddr{0xBD, 0xBD, 0xBD, 0xBD, 0xBD, 0xBD},

    }

    tcpLayer := &layers.TCP{

        SrcPort: layers.TCPPort\(4321\),

        DstPort: layers.TCPPort\(80\),

    }

    // And create the packet with the layers

    buffer = gopacket.NewSerializeBuffer\(\)

    gopacket.SerializeLayers\(buffer, options,

        ethernetLayer,

        ipLayer,

        tcpLayer,

        gopacket.Payload\(rawBytes\),

    \)

    outgoingPacket = buffer.Bytes\(\)

}

