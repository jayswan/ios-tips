# NetFlow Top-Talkers
In this post, keeping with the spirit of "quick-and-easy" ways to improve your productivity, we'll look at some of the CLI tools for use with NetFlow. My favorite one is the top talkers feature. How many times have you wondered what hosts or applications are using bandwidth on a link? The top talkers feature makes it easy to see. 

First, you need to activate NetFlow on the relevant interfaces. You do this by applying the "ip flow ingress" command, in interface configuration mode. 

```
test#sh run int f0/0.134 | i int|ip flow

interface FastEthernet0/0.134              
 ip flow ingress                           

```

In older IOS versions, the "ip flow ingress" command was "ip route-cache flow". This older version still works fine, but it's considered deprecated and the newer version is preferable. There's also an "ip flow egress" command that can be used with collectors that understand flow directionality.

Next, you enable to top talkers feature:

```
test#sh run | section top-talkers
ip flow-top-talkers
 top 10
 sort-by bytes
```

You can specify how many top talkers to cache, and the attribute by which to sort. Note the use of the "section" output modifier: we didn't discuss this in the article on output modifiers, but it's incredibly useful.

All you need to do from here is issue the "show ip flow top-talkers" command:

```
test#sh ip flow top-talkers
SrcIf         SrcIPaddress    DstIf         DstIPaddress   Pr SrcP DstP Bytes
Se0/0:1       10.116.147.101  Fa0/0.134     10.77.10.139   06 0050 05C0  8599K
Se0/0:1       10.116.147.101  Fa0/0.134*    10.77.10.139   06 0050 05C0  8599K
Se0/0:1       10.67.109.131   Fa0/0.134     10.77.10.127   06 0050 0D83  7119K
Se0/0:1       10.67.109.131   Fa0/0.134*    10.77.10.127   06 0050 0D83  7119K
Se0/0:1       10.116.147.117  Fa0/0.134     10.77.10.135   06 0050 0954  1584K
Se0/0:1       10.116.147.117  Fa0/0.134*    10.77.10.135   06 0050 0954  1584K
```

The output here is pretty self-explanatory. The only part that can be a little tricky is that the protocol, source port, and destination port fields are shown in hexadecimal. In this example, all of the flows are TCP flows; TCP is IP protocol 6. UDP flows would be listed as "11"; 11 in hex is 17 in base 10, and UDP is IP protocol 17.

The source port in each of these is 0x50 (that's short for hexadecimal 50), which converts to 80 in base 10. As you know, TCP port 80 is used for HTTP. Thus, in these flows we're seeing traffic from web servers at 10.116.147.101 and 10.67.109.131 going to three different HTTP clients. Strictly speaking, basic NetFlow is application-agnostic: we can't know for certain that this is actually HTTP traffic without looking at the application layer; it could be some other service running on port 80. Most likely, however, it's HTTP. Recent versions of IOS actually include the ability to export application-layer HTTP data with IPFIX, but that's beyond the scope of this article.

If you want to view the entire active NetFlow cache, use the "show ip cache flow" command. I find this particularly useful if I'm looking for smaller flows that might not show up in the top X talkers. The protocol and packet size statistics can also be handy if you're troubleshooting those types of problems:

```
testB#sh ip cache flow
IP packet size distribution (50401M total packets):   
1-32   64   96  128  160  192  224  256  288  320  352  384  416  448  480
.000 .296 .028 .088 .027 .015 .023 .016 .025 .004 .003 .004 .004 .004 .004    

512  544  576 1024 1536 2048 2560 3072 3584 4096 4608
.003 .074 .004 .018 .348 .000 .000 .000 .000 .000 .000
[output omitted]
Protocol         Total    Flows   Packets Bytes  Packets Active(Sec) Idle(Sec)
--------         Flows     /Sec     /Flow  /Pkt     /Sec     /Flow     /Flow
TCP-Telnet       84488      0.0         3    59      0.0       2.3      13.4
TCP-FTP         824292      0.1        14    61      2.8       3.5       4.1
TCP-FTPD       1152377      0.2        45   880     12.0       0.5       1.8
TCP-WWW      653021156    152.0        36   777   5610.5       4.2       7.2
TCP-SMTP     270501418     62.9         9   351    618.2       4.5       4.2
[output omitted]
GRE             164626      0.0        68   388      2.6      14.1      15.8
IP-other      14789920      3.4       288   449    993.4      49.9      15.4
Total:      2610739055    607.8        19   607  11734.9       2.5      12.2
SrcIf         SrcIPaddress    DstIf         DstIPaddress    Pr SrcP DstP  Pkts
Gi0/1.9       10.113.10.162   Gi0/0*          10.254.10.141 06 495F 7194   222 
[output omitted]
```

I deleted a bunch of output here for brevity, but you can see the main sections: a statistical distribution of packet sizes, a statistical distribution of common protocol types, and a raw list of entries in the NetFlow cache. The number of entries in the flow cache can be quite large; you'll definitely need to use output modifiers to filter this in a production environment.



