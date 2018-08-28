# 解码/构造数据包的例子

package main



import \(

    "fmt"

    "github.com/google/gopacket"

    "github.com/google/gopacket/layers"

\)



func main\(\) {

    // If we don't have a handle to a device or a file, but we have a bunch

    // of raw bytes, we can try to decode them in to packet information



    // NewPacket\(\) takes the raw bytes that make up the packet as the first parameter

    // The second parameter is the lowest level layer you want to decode. It will

    // decode that layer and all layers on top of it. The third layer

    // is the type of decoding: default\(all at once\), lazy\(on demand\), and NoCopy

    // which will not create a copy of the buffer



    // Create an packet with ethernet, IP, TCP, and payload layers

    // We are creating one we know will be decoded properly but

    // your byte source could be anything. If any of the packets

    // come back as nil, that means it could not decode it in to

    // the proper layer \(malformed or incorrect packet type\)

    payload := \[\]byte{2, 4, 6}

    options := gopacket.SerializeOptions{}

    buffer := gopacket.NewSerializeBuffer\(\)

    gopacket.SerializeLayers\(buffer, options,

        &layers.Ethernet{},

        &layers.IPv4{},

        &layers.TCP{},

        gopacket.Payload\(payload\),

    \)

    rawBytes := buffer.Bytes\(\)



    // Decode an ethernet packet

    ethPacket :=

        gopacket.NewPacket\(

            rawBytes,

            layers.LayerTypeEthernet,

            gopacket.Default,

        \)



    // with Lazy decoding it will only decode what it needs when it needs it

    // This is not concurrency safe. If using concurrency, use default

    ipPacket :=

        gopacket.NewPacket\(

            rawBytes,

            layers.LayerTypeIPv4,

            gopacket.Lazy,

        \)



    // With the NoCopy option, the underlying slices are referenced

    // directly and not copied. If the underlying bytes change so will

    // the packet

    tcpPacket :=

        gopacket.NewPacket\(

            rawBytes,

            layers.LayerTypeTCP,

            gopacket.NoCopy,

        \)



    fmt.Println\(ethPacket\)

    fmt.Println\(ipPacket\)

    fmt.Println\(tcpPacket\)

}



自定义协议

使用gopacket layer包不包含的协议。比如如果您要创建自己的l33t协议，甚至不使用TCP/IP或以太网，这是很有用的。



package main



import \(

    "fmt"

    "github.com/google/gopacket"

\)



// Create custom layer structure

type CustomLayer struct {

    // This layer just has two bytes at the front

    SomeByte    byte

    AnotherByte byte

    restOfData  \[\]byte

}



// Register the layer type so we can use it

// The first argument is an ID. Use negative

// or 2000+ for custom layers. It must be unique

var CustomLayerType = gopacket.RegisterLayerType\(

    2001,

    gopacket.LayerTypeMetadata{

        "CustomLayerType",

        gopacket.DecodeFunc\(decodeCustomLayer\),

    },

\)



// When we inquire about the type, what type of layer should

// we say it is? We want it to return our custom layer type

func \(l CustomLayer\) LayerType\(\) gopacket.LayerType {

    return CustomLayerType

}



// LayerContents returns the information that our layer

// provides. In this case it is a header layer so

// we return the header information

func \(l CustomLayer\) LayerContents\(\) \[\]byte {

    return \[\]byte{l.SomeByte, l.AnotherByte}

}



// LayerPayload returns the subsequent layer built

// on top of our layer or raw payload

func \(l CustomLayer\) LayerPayload\(\) \[\]byte {

    return l.restOfData

}



// Custom decode function. We can name it whatever we want

// but it should have the same arguments and return value

// When the layer is registered we tell it to use this decode function

func decodeCustomLayer\(data \[\]byte, p gopacket.PacketBuilder\) error {

    // AddLayer appends to the list of layers that the packet has

    p.AddLayer\(&CustomLayer{data\[0\], data\[1\], data\[2:\]}\)



    // The return value tells the packet what layer to expect

    // with the rest of the data. It could be another header layer,

    // nothing, or a payload layer.



    // nil means this is the last layer. No more decoding

    // return nil



    // Returning another layer type tells it to decode

    // the next layer with that layer's decoder function

    // return p.NextDecoder\(layers.LayerTypeEthernet\)



    // Returning payload type means the rest of the data

    // is raw payload. It will set the application layer

    // contents with the payload

    return p.NextDecoder\(gopacket.LayerTypePayload\)

}



func main\(\) {

    // If you create your own encoding and decoding you can essentially

    // create your own protocol or implement a protocol that is not

    // already defined in the layers package. In our example we are just

    // wrapping a normal ethernet packet with our own layer.

    // Creating your own protocol is good if you want to create

    // some obfuscated binary data type that was difficult for others

    // to decode



    // Finally, decode your packets:

    rawBytes := \[\]byte{0xF0, 0x0F, 65, 65, 66, 67, 68}

    packet := gopacket.NewPacket\(

        rawBytes,

        CustomLayerType,

        gopacket.Default,

    \)

    fmt.Println\("Created packet out of raw bytes."\)

    fmt.Println\(packet\)



    // Decode the packet as our custom layer

    customLayer := packet.Layer\(CustomLayerType\)

    if customLayer != nil {

        fmt.Println\("Packet was successfully decoded with custom layer decoder."\)

        customLayerContent, \_ := customLayer.\(\*CustomLayer\)

        // Now we can access the elements of the custom struct

        fmt.Println\("Payload: ", customLayerContent.LayerPayload\(\)\)

        fmt.Println\("SomeByte element:", customLayerContent.SomeByte\)

        fmt.Println\("AnotherByte element:", customLayerContent.AnotherByte\)

    }

}

更快地解码数据包 

如果我们知道我们要预期的得到的层，我们可以使用现有的结构来存储分组信息，而不是为每个需要时间和内存的分组创建新的结构。使用DecodingLayerParser更快。就像编组和解组数据一样。

package main



import \(

    "fmt"

    "github.com/google/gopacket"

    "github.com/google/gopacket/layers"

    "github.com/google/gopacket/pcap"

    "log"

    "time"

\)



var \(

    device       string = "eth0"

    snapshot\_len int32  = 1024

    promiscuous  bool   = false

    err          error

    timeout      time.Duration = 30 \* time.Second

    handle       \*pcap.Handle

    // Will reuse these for each packet

    ethLayer layers.Ethernet

    ipLayer  layers.IPv4

    tcpLayer layers.TCP

\)



func main\(\) {

    // Open device

    handle, err = pcap.OpenLive\(device, snapshot\_len, promiscuous, timeout\)

    if err != nil {

        log.Fatal\(err\)

    }

    defer handle.Close\(\)



    packetSource := gopacket.NewPacketSource\(handle, handle.LinkType\(\)\)

    for packet := range packetSource.Packets\(\) {

        parser := gopacket.NewDecodingLayerParser\(

            layers.LayerTypeEthernet,

            &ethLayer,

            &ipLayer,

            &tcpLayer,

        \)

        foundLayerTypes := \[\]gopacket.LayerType{}



        err := parser.DecodeLayers\(packet.Data\(\), &foundLayerTypes\)

        if err != nil {

            fmt.Println\("Trouble decoding layers: ", err\)

        }



        for \_, layerType := range foundLayerTypes {

            if layerType == layers.LayerTypeIPv4 {

                fmt.Println\("IPv4: ", ipLayer.SrcIP, "-&gt;", ipLayer.DstIP\)

            }

            if layerType == layers.LayerTypeTCP {

                fmt.Println\("TCP Port: ", tcpLayer.SrcPort, "-&gt;", tcpLayer.DstPort\)

                fmt.Println\("TCP SYN:", tcpLayer.SYN, " \| ACK:", tcpLayer.ACK\)

            }

        }

    }

}

