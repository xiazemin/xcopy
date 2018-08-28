# 抓取结果保存为PCAP格式文件

我们必须使用gapacket/pcapgo包来输出pcap的格式文件。



package main



import \(

    "fmt"

    "os"

    "time"



    "github.com/google/gopacket"

    "github.com/google/gopacket/layers"

    "github.com/google/gopacket/pcap"

    "github.com/google/gopacket/pcapgo"

\)



var \(

    deviceName  string = "eth0"

    snapshotLen int32  = 1024

    promiscuous bool   = false

    err         error

    timeout     time.Duration = -1 \* time.Second

    handle      \*pcap.Handle

    packetCount int = 0

\)



func main\(\) {

    // Open output pcap file and write header 

    f, \_ := os.Create\("test.pcap"\)

    w := pcapgo.NewWriter\(f\)

    w.WriteFileHeader\(snapshotLen, layers.LinkTypeEthernet\)

    defer f.Close\(\)



    // Open the device for capturing

    handle, err = pcap.OpenLive\(deviceName, snapshotLen, promiscuous, timeout\)

    if err != nil {

        fmt.Printf\("Error opening device %s: %v", deviceName, err\)

        os.Exit\(1\)

    }

    defer handle.Close\(\)



    // Start processing packets

    packetSource := gopacket.NewPacketSource\(handle, handle.LinkType\(\)\)

    for packet := range packetSource.Packets\(\) {

        // Process packet here

        fmt.Println\(packet\)

        w.WritePacket\(packet.Metadata\(\).CaptureInfo, packet.Data\(\)\)

        packetCount++



        // Only capture 100 and then stop

        if packetCount &gt; 100 {

            break

        }

    }

}

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

读取PCAP格式文件来查看分析网络数据包

我们可以使用tcpdump创建的文件。



\# Capture packets to test.pcap file

sudo tcpdump -w test.pcap

package main



// Use tcpdump to create a test file

// tcpdump -w test.pcap

// or use the example above for writing pcap files



import \(

    "fmt"

    "github.com/google/gopacket"

    "github.com/google/gopacket/pcap"

    "log"

\)



var \(

    pcapFile string = "test.pcap"

    handle   \*pcap.Handle

    err      error

\)



func main\(\) {

    // Open file instead of device

    handle, err = pcap.OpenOffline\(pcapFile\)

    if err != nil { log.Fatal\(err\) }

    defer handle.Close\(\)



    // Loop through packets in file

    packetSource := gopacket.NewPacketSource\(handle, handle.LinkType\(\)\)

    for packet := range packetSource.Packets\(\) {

        fmt.Println\(packet\)

    }

}

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

设置过滤器

\#只抓取TCP协议80端口的数据

package main



import \(

    "fmt"

    "github.com/google/gopacket"

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

\)



func main\(\) {

    // Open device

    handle, err = pcap.OpenLive\(device, snapshot\_len, promiscuous, timeout\)

    if err != nil {

        log.Fatal\(err\)

    }

    defer handle.Close\(\)



    // Set filter

    var filter string = "tcp and port 80"

    err = handle.SetBPFFilter\(filter\)

    if err != nil {

        log.Fatal\(err\)

    }

    fmt.Println\("Only capturing TCP port 80 packets."\)



    packetSource := gopacket.NewPacketSource\(handle, handle.LinkType\(\)\)

    for packet := range packetSource.Packets\(\) {

        // Do something with a packet here.

        fmt.Println\(packet\)

    }



}

