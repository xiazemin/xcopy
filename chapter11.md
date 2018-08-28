# gopacket包

使用go语言的gopacket包可以非常容易地实现抓包，gopacket包构建在libpcap的之上。下面介绍下gopacket的常见用法。

\# Get the gopacket package from GitHub

go get github.com/google/gopacket

\# Pcap dev headers might be necessary

sudo apt-get install libpcap-dev

获取所有的网络设备信息

package main

import \(

```
"fmt"

"log"

"github.com/google/gopacket/pcap"
```

\)

func main\(\) {

```
// Find all devices

devices, err := pcap.FindAllDevs\(\)

if err != nil {

    log.Fatal\(err\)

}



// Print device information

fmt.Println\("Devices found:"\)

for \_, device := range devices {

    fmt.Println\("\nName: ", device.Name\)

    fmt.Println\("Description: ", device.Description\)

    fmt.Println\("Devices addresses: ", device.Description\)

    for \_, address := range device.Addresses {

        fmt.Println\("- IP address: ", address.IP\)

        fmt.Println\("- Subnet mask: ", address.Netmask\)

    }

}
```

}



打开设备实时捕捉

package main

import \(

```
"fmt"

"github.com/google/gopacket"

"github.com/google/gopacket/pcap"

"log"

"time"
```

\)

var \(

```
device       string = "eth0"

snapshot\_len int32  = 1024

promiscuous  bool   = false

err          error

timeout      time.Duration = 30 \* time.Second

handle       \*pcap.Handle
```

\)

func main\(\) {

```
// Open device

handle, err = pcap.OpenLive\(device, snapshot\_len, promiscuous, timeout\)

if err != nil {log.Fatal\(err\) }

defer handle.Close\(\)



// Use the handle as a packet source to process all packets

packetSource := gopacket.NewPacketSource\(handle, handle.LinkType\(\)\)

for packet := range packetSource.Packets\(\) {

    // Process packet here

    fmt.Println\(packet\)

}
```

}

