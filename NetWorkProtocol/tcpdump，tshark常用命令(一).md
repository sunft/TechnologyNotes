@[TOC](tcpdump，tshark常用命令)
# 抓包工具简介

工作过一段时间的后台程序员一般都知道Linux命令行抓包使用**tcpdump**，而windows上一般使用**Wireshark、Fiddler**和**Charles**等带有图形界面的工具来抓包。

实际上在windows平台上也有对应的命令行抓包工具，例如tcpdump的windows版本**WinDump**，wireshark的命令行工具**tshark**等。这篇文章主要说一说tcpdump和tshark这两个命令行工具的基本使用，相对简单，其他的命令选项以及过滤特定的协议、基本的分析放在下一篇文章说明。对比讲解，方便比较差异点。这里使用Linux版本是Centos 7，windows版本是windoes10。

## 1、查看命令使用方法
### tcpdump代码示例
使用-h选项查看基本的命令使用帮助。如果需要查看详细的使用方法可以使用**man tcpdump**命令查看，也可以上官网查看。

```bash
[sunft@localhost ~]$ tcpdump -h
tcpdump version 4.9.0
libpcap version 1.5.3
OpenSSL 1.0.1e-fips 11 Feb 2013
Usage: tcpdump [-aAbdDefhHIJKlLnNOpqStuUvxX#] [ -B size ] [ -c count ]
		[ -C file_size ] [ -E algo:secret ] [ -F file ] [ -G seconds ]
		[ -i interface ] [ -j tstamptype ] [ -M secret ] [ --number ]
		[ -Q|-P in|out|inout ]
		[ -r file ] [ -s snaplen ] [ --time-stamp-precision precision ]
		[ --immediate-mode ] [ -T type ] [ --version ] [ -V file ]
		[ -w file ] [ -W filecount ] [ -y datalinktype ] [ -z postrotate-command ]
		[ -Z user ] [ expression ]
[sunft@localhost ~]$ 
```

### tshark代码示例
因为选项比较长，这里只是截取部分代码。

```bash
C:\Users\sunft>tshark -h
TShark (Wireshark) 3.0.5 (v3.0.5-0-g752a55954770)
Dump and analyze network traffic.
See https://www.wireshark.org for more information.

Usage: tshark [options] ...

Capture interface:
  -i <interface>           name or idx of interface (def: first non-loopback)
  -f <capture filter>      packet filter in libpcap filter syntax
  -s <snaplen>             packet snapshot length (def: appropriate maximum)
  -p                       don't capture in promiscuous mode
  -I                       capture in monitor mode, if available
  -B <buffer size>         size of kernel buffer (def: 2MB)
  -y <link type>           link layer type (def: first appropriate)
  --time-stamp-type <type> timestamp method for interface
  -D                       print list of interfaces and exit
  -L                       print list of link-layer types of iface and exit
  --list-time-stamp-types  print list of timestamp types for iface and exit
```

## 2、列出所有可用网卡
### tcpdump代码示例
 查看网卡的方式有很多种，也可以使用其他方式。这里只列举出关键代码。
 
```bash
[sunft@localhost ~]$ ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.248.134  netmask 255.255.255.0  broadcast 192.168.248.255
        inet6 fe80::176e:36f2:18ab:c561  prefixlen 64  scopeid 0x20<link>
        inet6 fd15:4ba5:5a2b:1008:c7f1:b0a6:ecf:6bfc  prefixlen 64  scopeid 0x0<global>
        ether 00:0c:29:59:4f:1a  txqueuelen 1000  (Ethernet)
        RX packets 819077  bytes 1148807591 (1.0 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 314623  bytes 19075992 (18.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
### tshark代码示例
如果安装了tshark，直接加上-D参数即可查看网卡。也可以使用ipconfig等命令查看。

```bash
C:\Users\sunft>tshark -D
1. {0B1F7D4B-66C5-4236-8E1E-84F962B90027} (Ethernet)
2. {29A71B75-0F56-41AF-B08D-9AF68A77ADC6} (Wi-Fi)
3. {64AC4577-5CD1-4B21-975D-7C60B6E70318} (VMware Network Adapter VMnet1)
4. {F603BF38-10C4-45A4-9A2D-F6D03993269E} (Bluetooth Network Connection)
5. {683A6B15-6954-4C49-8E18-C098A8AA3EE2} (VMware Network Adapter VMnet8)
6. {2010E882-0928-44EF-9126-E10EDC49D042} (Local Area Connection* 2)
7. {18433062-232D-4CC2-99B2-251419BD524D} (Local Area Connection* 3)
```
## 3、捕获指定网卡的网络包
### tcpdump代码示例
-i参数用于指定捕获哪个网卡的网络数据包，这里使用的是虚拟机，需要root权限，按<kbd>Ctrl</kbd> + <kbd>c</kbd>结束抓取，可以看到有11个包被捕获，并且直接被输出在屏幕上。

```bash
[sunft@localhost ~]$ tcpdump -i ens33
tcpdump: ens33: You don't have permission to capture on that device
(socket: Operation not permitted)
[sunft@localhost ~]$ sudo su
[sudo] sunft 的密码：
[root@localhost sunft]# tcpdump -i ens33
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes
20:51:30.977758 IP6 gateway > ff02::1: ICMP6, router advertisement, length 80
20:51:30.979641 IP localhost.36233 > bogon.domain: 25666+ PTR? 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.2.0.f.f.ip6.arpa. (90)
20:51:33.236398 IP bogon.domain > localhost.36233: 25666 NXDomain 0/1/0 (154)
20:51:33.293010 IP localhost.20380 > bogon.domain: 38132+ PTR? 2.2.2.2.0.c.e.f.f.f.6.5.0.5.2.0.0.0.0.0.0.0.0.0.0.0.0.0.0.8.e.f.ip6.arpa. (90)
20:51:35.298239 IP bogon.domain > localhost.20380: 38132 NXDomain 0/1/0 (139)
20:51:35.299515 IP localhost.53782 > bogon.domain: 53444+ PTR? 2.248.168.192.in-addr.arpa. (44)
20:51:35.993796 ARP, Request who-has bogon tell localhost, length 28
20:51:35.994074 ARP, Reply bogon is-at 00:50:56:f6:24:d8 (oui Unknown), length 46
20:51:37.305235 ARP, Request who-has localhost tell bogon, length 46
20:51:37.305267 ARP, Reply localhost is-at 00:0c:29:59:4f:1a (oui Unknown), length 28
20:51:37.305583 IP bogon.domain > localhost.53782: 53444 1/0/0 PTR bogon. (63)
^C
11 packets captured
11 packets received by filter
0 packets dropped by kernel
```
### tshark代码示例
-i参数用于指定网卡，与tcpdump不同的是，后面直接接数字，而不是网卡名称，指定捕获流经第几个网卡的网络包，可以看到有17个包被捕获。

```bash
C:\Users\sunft>tshark -i 1
The NPF driver isn't running.  You may have trouble capturing or
listing interfaces.
Capturing on 'Ethernet'
    1   0.000000 192.168.1.101 → 36.99.30.204 TCP 55 60665 → 80 [ACK] Seq=1 Ack=1 Win=507 Len=1
    2   0.029671 36.99.30.204 → 192.168.1.101 TCP 60 80 → 60665 [ACK] Seq=1 Ack=2 Win=501 Len=0
    3   0.483049 192.168.1.101 → 129.204.167.254 TCP 64 61522 → 31077 [PSH, ACK] Seq=1 Ack=1 Win=256 Len=10
    4   0.496438 129.204.167.254 → 192.168.1.101 TCP 60 31077 → 61522 [ACK] Seq=1 Ack=11 Win=239 Len=0
    5   0.855910 192.168.1.101 → 10.195.1.1   DNS 73 Standard query 0x703a A p3p.sogou.com
    6   0.857636   10.195.1.1 → 192.168.1.101 DNS 121 Standard query response 0x703a A p3p.sogou.com A 211.159.235.96 A 211.159.235.30 A 211.159.235.100
    7   0.858949 192.168.1.101 → 211.159.235.96 TCP 66 62682 → 80 [SYN] Seq=0 Win=65535 Len=0 MSS=1460 WS=256 SACK_PERM=1
    8   0.899890 211.159.235.96 → 192.168.1.101 TCP 66 80 → 62682 [SYN, ACK] Seq=0 Ack=1 Win=29200 Len=0 MSS=1350 SACK_PERM=1 WS=256
    9   0.900145 192.168.1.101 → 211.159.235.96 TCP 54 62682 → 80 [ACK] Seq=1 Ack=1 Win=262144 Len=0
   10   0.900835 192.168.1.101 → 211.159.235.96 HTTP 447 GET /seupdater.gif?h=0B5DCFBD8C9D700F2C248838C439275E&elapse=161564000&res=0 HTTP/1.1
   11   0.942755 211.159.235.96 → 192.168.1.101 TCP 60 80 → 62682 [ACK] Seq=1 Ack=394 Win=30464 Len=0
   12   0.946162 211.159.235.96 → 192.168.1.101 HTTP 193 HTTP/1.1 200 OK
   13   0.946387 192.168.1.101 → 211.159.235.96 TCP 54 62682 → 80 [ACK] Seq=394 Ack=140 Win=261888 Len=0
   14   1.703680 117.18.232.240 → 192.168.1.101 TCP 1404 80 → 62582 [PSH, ACK] Seq=1 Ack=1 Win=294 Len=1350
   15   1.743695 192.168.1.101 → 117.18.232.240 TCP 54 62582 → 80 [ACK] Seq=1 Ack=1351 Win=258 Len=0
   16   3.715851 111.13.94.52 → 192.168.1.101 TCP 60 80 → 62638 [FIN, ACK] Seq=1 Ack=1 Win=119 Len=0
   17   3.716200 192.168.1.101 → 111.13.94.52 TCP 54 62638 → 80 [ACK] Seq=1 Ack=2 Win=1022 Len=0
17 packets captured
```
## 4、将抓到的包保存到指定的文件
### tcpdump代码示例
使用-w参数指定文件名，文件默认保存在当前目录下。按ctrl+c终止抓包。可以看到一共抓到220个包。

```bash
[root@localhost sunft]# tcpdump -i ens33 -w packets.pcap
tcpdump: listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes
^C220 packets captured
220 packets received by filter
0 packets dropped by kernel

```
### tshark代码示例
使用-i参数指定网卡，-w参数指定要写入的文件名称。最后可以看到被抓到的包已经被列出来了，可以看到有24个包被抓到。

```bash
I:\packet>tshark -i 1 -w packets.pcap
The NPF driver isn't running.  You may have trouble capturing or
listing interfaces.
Capturing on 'Ethernet'
24

I:\packet>dir
 Volume in drive I is 学习
 Volume Serial Number is 0003-7EAF

 Directory of I:\packet

2019/10/15  00:38    <DIR>          .
2019/10/15  00:38    <DIR>          ..
2019/10/16  21:04             5,812 packets.pcap
               1 File(s)          5,812 bytes
               2 Dir(s)  41,436,913,664 bytes free
```
## 5、读取抓到的包
### tcpdump代码示例
使用-r参数读取对应的抓到的包，如果不指定路径，默认读取当前路径下的包。最前面的列是时间，后面是协议以及其他一些信息，这些在下一篇文章中进行详细分析。

```bash
[root@localhost sunft]# tcpdump -r packets.pcap 
reading from file packets.pcap, link-type EN10MB (Ethernet)
20:59:00.987471 IP6 gateway > ff02::1: ICMP6, router advertisement, length 80
20:59:10.236372 IP6 fe80::24b1:28c6:e76:62f > ff02::1:ffc0:2222: ICMP6, neighbor solicitation, who has gateway, length 32
20:59:11.027243 IP6 fe80::24b1:28c6:e76:62f > ff02::1:ffc0:2222: ICMP6, neighbor solicitation, who has gateway, length 32
20:59:12.026918 IP6 fe80::24b1:28c6:e76:62f > ff02::1:ffc0:2222: ICMP6, neighbor solicitation, who has gateway, length 32
20:59:23.342767 IP localhost.45842 > 119.28.183.184.ntp: NTPv4, Client, length 48
20:59:23.362555 IP 119.28.183.184.ntp > localhost.45842: NTPv4, Server, length 48
20:59:27.741194 IP localhost.58323 > a.chl.la.ntp: NTPv4, Client, length 48
20:59:28.345961 ARP, Request who-has bogon tell localhost, length 28
```
### tshark代码示例
使用-r参数读取包，如果不指定路径，默认读取当前路径下的包。第一列是序号，第二列表示第几秒抓到包。

```bash
I:\packet>tshark -r packets.pcap
    1   0.000000 192.168.1.101 → 180.163.243.187 TCP 55 62559 → 80 [ACK] Seq=1 Ack=1 Win=510 Len=1
    2   0.031258 180.163.243.187 → 192.168.1.101 TCP 60 80 → 62559 [ACK] Seq=1 Ack=2 Win=131 Len=0
    3   1.497118 192.168.1.101 → 13.226.120.111 TLSv1.2 100 Application Data
    4   1.679906 13.226.120.111 → 192.168.1.101 TLSv1.2 100 Application Data
    5   1.720486 192.168.1.101 → 13.226.120.111 TCP 54 62735 → 443 [ACK] Seq=47 Ack=47 Win=258 Len=0
    6   3.119496 192.168.1.101 → 129.204.167.254 TCP 64 61522 → 31077 [PSH, ACK] Seq=1 Ack=1 Win=255 Len=10
    7   3.132354 129.204.167.254 → 192.168.1.101 TCP 60 31077 → 61522 [ACK] Seq=1 Ack=11 Win=239 Len=0
    8   5.597833 192.168.1.101 → 60.28.219.13 TCP 66 62769 → 80 [SYN] Seq=0 Win=65535 Len=0 MSS=1460 WS=256 SACK_PERM=1
    9   5.640581 60.28.219.13 → 192.168.1.101 TCP 66 80 → 62769 [SYN, ACK] Seq=0 Ack=1 Win=14600 Len=0 MSS=1350 SACK_PERM=1 WS=512
```
## 6、读取指定个数的包
### tcpdump代码示例
使用-c参数指定读取包的个数，这里读取5个包，下面恰好输出五行。

```bash
[root@localhost sunft]# tcpdump -r packets.pcap -c5
reading from file packets.pcap, link-type EN10MB (Ethernet)
20:59:00.987471 IP6 gateway > ff02::1: ICMP6, router advertisement, length 80
20:59:10.236372 IP6 fe80::24b1:28c6:e76:62f > ff02::1:ffc0:2222: ICMP6, neighbor solicitation, who has gateway, length 32
20:59:11.027243 IP6 fe80::24b1:28c6:e76:62f > ff02::1:ffc0:2222: ICMP6, neighbor solicitation, who has gateway, length 32
20:59:12.026918 IP6 fe80::24b1:28c6:e76:62f > ff02::1:ffc0:2222: ICMP6, neighbor solicitation, who has gateway, length 32
20:59:23.342767 IP localhost.45842 > 119.28.183.184.ntp: NTPv4, Client, length 48

```

### tshark代码示例
与tcpdump一样，同样使用-c参数指定读取的包的个数。这里读取5个包。

```bash
I:\packet>tshark -r packets.pcap -c5
    1   0.000000 192.168.1.101 → 180.163.243.187 TCP 55 62559 → 80 [ACK] Seq=1 Ack=1 Win=510 Len=1
    2   0.031258 180.163.243.187 → 192.168.1.101 TCP 60 80 → 62559 [ACK] Seq=1 Ack=2 Win=131 Len=0
    3   1.497118 192.168.1.101 → 13.226.120.111 TLSv1.2 100 Application Data
    4   1.679906 13.226.120.111 → 192.168.1.101 TLSv1.2 100 Application Data
    5   1.720486 192.168.1.101 → 13.226.120.111 TCP 54 62735 → 443 [ACK] Seq=47 Ack=47 Win=258 Len=0
```
## 7、抓取指定个数的包
### tcpdump代码示例
使用-c参数指定抓取包的个数，这里抓取10个包。

```bash
[root@localhost sunft]# tcpdump -i ens33 -w packets.pcap -c10
tcpdump: listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes
10 packets captured
10 packets received by filter
0 packets dropped by kernel
[root@localhost sunft]# 
```
### tshark代码示例
这里抓取10个包后保存在packets.pcap文件中，抓到指定个数的包后会自动停止运行。

```bash
I:\packet>tshark -i 1 -w packets.pcap -c10
The NPF driver isn't running.  You may have trouble capturing or
listing interfaces.
Capturing on 'Ethernet'
10

I:\packet>
```
## 8、读取第一个包的详细信息
### tcpdump代码示例
使用-v参数表示读取详细信息，注意这里的v是小写。

```bash
[root@localhost sunft]# tcpdump -r packets.pcap -c1 -v
reading from file packets.pcap, link-type EN10MB (Ethernet)
21:14:31.004981 IP6 (hlim 255, next-header ICMPv6 (58) payload length: 80) gateway > ff02::1: [icmp6 sum ok] ICMP6, router advertisement, length 80
	hop limit 64, Flags [none], pref medium, router lifetime 3000s, reachable time 0ms, retrans time 0ms
	  prefix info option (3), length 32 (4): fd15:4ba5:5a2b:1008::/64, Flags [onlink, auto, router], valid time 86400s, pref. time 14400s
	  source link-address option (1), length 8 (1): 00:50:56:f6:24:d8
	  route info option (24), length 24 (3):  fd15:4ba5:5a2b:1008::2222/64, pref=medium, lifetime=90000s
[root@localhost sunft]# 
```
### tshark代码示例
使用-V参数读取详细信息，这里的V是大写，这里的展示的信息已经十分详细了，下面的输出省略了十六进制表示的形式，后续再出文章进行详细分析。

```bash
I:\packet>tshark -r packets.pcap -c1 -V
Frame 1: 1404 bytes on wire (11232 bits), 1404 bytes captured (11232 bits) on interface 0
    Interface id: 0 ({0B1F7D4B-66C5-4236-8E1E-84F962B90027})
        Interface name: {0B1F7D4B-66C5-4236-8E1E-84F962B90027}
        Interface description: Ethernet
    Encapsulation type: Ethernet (1)
    Arrival Time: Oct 16, 2019 21:16:18.788654000 China Standard Time
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1571231778.788654000 seconds
    [Time delta from previous captured frame: 0.000000000 seconds]
    [Time delta from previous displayed frame: 0.000000000 seconds]
    [Time since reference or first frame: 0.000000000 seconds]
    Frame Number: 1
    Frame Length: 1404 bytes (11232 bits)
    Capture Length: 1404 bytes (11232 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: eth:ethertype:ip:tcp:data]
Ethernet II, Src: Tp-LinkT_08:f0:0c (50:bd:5f:08:f0:0c), Dst: LcfcHefe_6e:a6:c9 (54:e1:ad:6e:a6:c9)
    Destination: LcfcHefe_6e:a6:c9 (54:e1:ad:6e:a6:c9)
        Address: LcfcHefe_6e:a6:c9 (54:e1:ad:6e:a6:c9)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Source: Tp-LinkT_08:f0:0c (50:bd:5f:08:f0:0c)
        Address: Tp-LinkT_08:f0:0c (50:bd:5f:08:f0:0c)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Type: IPv4 (0x0800)
Internet Protocol Version 4, Src: 35.220.188.54, Dst: 192.168.1.101
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
        0000 00.. = Differentiated Services Codepoint: Default (0)
        .... ..00 = Explicit Congestion Notification: Not ECN-Capable Transport (0)
    Total Length: 1390
    Identification: 0xd2b1 (53937)
    Flags: 0x4000, Don't fragment
        0... .... .... .... = Reserved bit: Not set
        .1.. .... .... .... = Don't fragment: Set
        ..0. .... .... .... = More fragments: Not set
        ...0 0000 0000 0000 = Fragment offset: 0
    Time to live: 61
    Protocol: TCP (6)
    Header checksum: 0xc3b8 [validation disabled]
    [Header checksum status: Unverified]
    Source: 35.220.188.54
    Destination: 192.168.1.101
Transmission Control Protocol, Src Port: 43335, Dst Port: 62781, Seq: 1, Ack: 1, Len: 1350
    Source Port: 43335
    Destination Port: 62781
    [Stream index: 0]
    [TCP Segment Len: 1350]
    Sequence number: 1    (relative sequence number)
    [Next sequence number: 1351    (relative sequence number)]
    Acknowledgment number: 1    (relative ack number)
    0101 .... = Header Length: 20 bytes (5)
    Flags: 0x010 (ACK)
        000. .... .... = Reserved: Not set
        ...0 .... .... = Nonce: Not set
        .... 0... .... = Congestion Window Reduced (CWR): Not set
        .... .0.. .... = ECN-Echo: Not set
        .... ..0. .... = Urgent: Not set
        .... ...1 .... = Acknowledgment: Set
        .... .... 0... = Push: Not set
        .... .... .0.. = Reset: Not set
        .... .... ..0. = Syn: Not set
        .... .... ...0 = Fin: Not set
        [TCP Flags: ·······A····]
    Window size value: 239
    [Calculated window size: 239]
    [Window size scaling factor: -1 (unknown)]
    Checksum: 0x0aad [unverified]
    [Checksum Status: Unverified]
    Urgent pointer: 0
    [SEQ/ACK analysis]
        [Bytes in flight: 1350]
        [Bytes sent since last PSH flag: 1350]
    [Timestamps]
        [Time since first frame in this TCP stream: 0.000000000 seconds]
        [Time since previous frame in this TCP stream: 0.000000000 seconds]
    TCP payload (1350 bytes)
Data (1350 bytes)
```
## 9、查看包的16进制形式和ASCII码的形式
### tcpdump代码示例
使用-X参数将包以16进制和ASCII码的形式显示出来，注意这里的X是大写。

```bash
[root@localhost sunft]# tcpdump -Xr packets.pcap
reading from file packets.pcap, link-type EN10MB (Ethernet)
21:14:31.004981 IP6 gateway > ff02::1: ICMP6, router advertisement, length 80
	0x0000:  6000 0000 0050 3aff fe80 0000 0000 0000  `....P:.........
	0x0010:  0250 56ff fec0 2222 ff02 0000 0000 0000  .PV...""........
	0x0020:  0000 0000 0000 0001 8600 2bac 4000 0bb8  ..........+.@...
	0x0030:  0000 0000 0000 0000 0304 40e0 0001 5180  ..........@...Q.
	0x0040:  0000 3840 0000 0000 fd15 4ba5 5a2b 1008  ..8@......K.Z+..
	0x0050:  0000 0000 0000 0000 0101 0050 56f6 24d8  ...........PV.$.
	0x0060:  1803 4000 0001 5f90 fd15 4ba5 5a2b 1008  ..@..._...K.Z+..
	0x0070:  0000 0000 0000 2222                      ......""

```
### tshark代码示例
使用-x参数将包以16进制和ASCII码的形式显示出来，注意这里的x是小写。下面只是截取了部分代码，十六进制不容易看懂也不太容易理解，后面出文章进行说明。

```bash
I:\packet>tshark -xr packets.pcap -c1
0000  54 e1 ad 6e a6 c9 50 bd 5f 08 f0 0c 08 00 45 00   T..n..P._.....E.
0010  05 6e d2 b1 40 00 3d 06 c3 b8 23 dc bc 36 c0 a8   .n..@.=...#..6..
0020  01 65 a9 47 f5 3d 73 b5 37 cb 10 c5 01 22 50 10   .e.G.=s.7...."P.
0030  00 ef 0a ad 00 00 ce cb 19 52 1a d1 f1 76 1b df   .........R...v..
0040  30 7a e0 34 11 f9 49 15 9e c4 31 48 c7 3b 67 d7   0z.4..I...1H.;g.
0050  9d f7 59 fd 4d 39 28 9f 2d d7 66 d9 f1 24 fb da   ..Y.M9(.-.f..$..
0060  6a 4f 5b 54 95 83 8b c4 77 e3 61 11 86 10 01 13   jO[T....w.a.....
0070  b2 78 6c 16 e2 c2 da cb 19 de b6 a8 ae ad db ac   .xl.............
0080  fe 80 de 2e 23 9e fa 3f 00 3d 4c 08 40 19 f1 b6   ....#..?.=L.@...
0090  29 1c 5f 3b ba 52 d3 30 e0 24 59 99 b8 2e 94 df   )._;.R.0.$Y.....
00a0  a2 4c 77 ba 37 ee c2 d5 77 86 e1 a5 66 e4 78 29   .Lw.7...w...f.x)
00b0  fa 1e 4c a1 36 e3 4d 58 e7 9d 91 0e 2a 97 ab 13   ..L.6.MX....*...
00c0  8a e7 73 f8 59 f2 17 ac 09 94 5e f7 9c 7c d2 1b   ..s.Y.....^..|..
00d0  16 69 08 c4 8f d5 7a a3 ac e3 71 7f f1 92 77 78   .i....z...q...wx
```
## 10、指定想要抓到的字节数
在有些操作系统中，默认只抓每个帧的前96个字节，我们可以用“-s”参数来指定想要抓到的字节数，比如下面的例子抓取每个包的前1000字节。
### tcpdump代码示例
使用-s参数指定每个包抓取多少个字节。

```bash
[root@localhost sunft]# tcpdump -i ens33 -s 1000 -w packets.pcap -c10
tcpdump: listening on ens33, link-type EN10MB (Ethernet), capture size 1000 bytes
10 packets captured
10 packets received by filter
0 packets dropped by kernel
[root@localhost sunft]# 
```
### tshark代码示例
使用-s参数指定每个包抓取多少个字节。

```bash
I:\packet>tshark -i 1 -s 1000 -w packets.pcap -c10
The NPF driver isn't running.  You may have trouble capturing or
listing interfaces.
Capturing on 'Ethernet'
10

I:\packet>
```

**以上10条命令中属于比较基础的命令，相对来说比较少见的是第10条命令，一般的博客不会写。**

**参考材料：**
《Practical Packet analysis, 3rd edition》
《Wireshark网络分析的艺术》
《Wireshark网络分析就是这么简单》


_下面是我的个人技术公众号，欢迎关注！_
![个人微信号](https://img-blog.csdnimg.cn/20191016213547817.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JydWNlTGVlTnVtYmVyT25l,size_16,color_FFFFFF,t_70) 


