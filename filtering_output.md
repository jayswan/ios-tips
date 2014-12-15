#Filtering Output


##Show Only Interfaces with Assigned Addresses

The `show ip interface brief` command is ubiquitous in almost everyone's IOS arsenal. On a platform with a lot of interfaces, though, many of those interfaces will frequently not have IP addresses assigned to them. Layer 3 switches and PRI gateways are two particularly common examples where this happens. You can filter out all of the unassigned interfaces with a simple exclusion filter, leaving only the ones that are likely to be interesting. Specifically, we use the `exclude` output modifier to remove lines with the "unassigned" word from the display.


### Before
```
Switch#show ip interface brief
Interface IP-Address OK? Method Status Protocol
Vlan1 unassigned YES unset administratively down down
FastEthernet1/0/1 10.1.1.1 YES manual down down
FastEthernet1/0/2 unassigned YES unset down down
FastEthernet1/0/3 unassigned YES unset down down
FastEthernet1/0/4 unassigned YES unset down down
```

###After
```
Switch#sh ip int b | exclude un
Interface IP-Address OK? Method Status Protocol
FastEthernet1/0/1 10.1.1.1 YES manual down down
```

##Show Only Connected Switch Interfaces

In this case, we'll use an exclusion filter to remove lines with "not" in them, thus removing all interfaces from the display that are in the "notconnect" state. Note that I've shortened "exclude" to "e" as a time-saver.

###Before
```
Test#sh int status

Port      Name               Status       Vlan       Duplex  Speed Type 
Gi1/0/1   Test               connected    64         a-full  a-100 10/100/1000BaseTX
Gi1/0/2   Test      		 notconnect   65           auto   auto 10/100/1000BaseTX
Gi1/0/3   Test               connected    64         a-full  a-100 10/100/1000BaseTX
```

###After
```
Test#sh int status | e not

Port      Name               Status       Vlan       Duplex  Speed Type 
Gi1/0/1   Test               connected    64         a-full  a-100 10/100/1000BaseTX
Gi1/0/3   Test               connected    64         a-full  a-100 10/100/1000BaseTX
```

##Incorporating Regular Expressions

When I was teaching Cisco classes, many students were annoyed that there's no built-in command that displays a list of all interface names and IP addresses along with the relevant subnet mask. You can use a regular expression to get around this annoyance:

```
#sh ip int | i line|/..$ 
Vlan1 is administratively down, line protocol is down
Vlan64 is up, line protocol is up
  Internet address is 10.30.64.1/24
Vlan65 is up, line protocol is up
  Internet address is 10.30.65.1/24
```

I'm not going to go into a detailed explanation of how regular expressions work--there are many good tutorials available via a web search, as well as several good books on the subject. Suffice to say that the command expression above translates into English like this:"take the 'show ip interface' command, and from the output include only lines that include the string "line" OR a string that ends with a "/" character followed by two characters."The result is that we see the interface name for all interfaces, followed by the IP address and mask in CIDR format. It's not perfect: for example, it will display all interface names, even if there's no address assigned to them. Also, it shows interfaces that are down, which might not be desirable. We might come up with workarounds for these problems by making different or more complex regular expressions, but the point of this blog is to show quick, easy-to-remember tricks that will help you in day-to-day operations, even at the expense of imperfections. 

##Other Handy Ideas
Here are a couple of other favorites of mine:

###Show Only Non-Phone CDP Neighbors
```
TEST-sw#sh cdp n | e SEP178           T     AIR-LAP113 Fas 0
```

Since Cisco IP phones have a device ID that begins with "SEP" (which stand for "Selsius Ethernet Phone", by the way), this is a quick way to get only non-phone neighbors, which is handy on a large IP telephony access switch.

###Show Access Port Status for a single VLAN
```
Test-sw#sh int status | i _64_
Gi1/0/1                      connected    64         a-full  a-100 10/100/1000BaseTX
```

###Show Access Port Status for Two VLANs
```
Test-sw#sh int status | i _64_|_76_
Gi1/0/1                      connected    64         a-full  a-100 10/100/1000BaseTX
Gi1/0/17  PC/Phone           notconnect   76           auto   auto 10/100/1000BaseTX
```

###Find Unused Switch Ports
```
Test-sw#sh int counter | i 0 + 0 + 0 + 0
Gi2/0/45               0              0              0              0 
Gi2/0/46               0              0              0              0 
Gi2/0/48               0              0              0              0 
Gi2/1/1                0              0              0              0
```

Note: there are better ways to get this to work in pure regex, but I couldn't make them work in IOS.

