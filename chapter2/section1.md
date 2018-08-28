# tcpreplay-edit

tcpreplay-edit 实时修改包数据并回放，它是将 tcprewrite 和 tcpreplay 用一条命令实现。其好处是修改包数据不会新生成 pcap 文件。如果是需要不断的改写一个包文件并回放建议使用 tcpreplay-edit，如果是需要一次改写一个包文件并多次回放建议使用 tcprewrite 和 tcpreplay 的结合，这样具有更好的回放速率。 





　　tcpreplay-edit小例子



　　编写脚本，不断改写包文件的 IP 地址并回放： 



　　比如，你自己写个test.sh脚本，然后赋予chmod 777 test.sh。执行就是



for i in {1..255}

do

tcpreplay-edit --endpoints=1.1.2.$i:1.1.1.2 -t -i eth2 -I eth1 -c edit.cach edit.pcap

done

 







