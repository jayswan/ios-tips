# Finding IP Addresses
This is a fairly basic task in networking, yet for whatever reason it's something that often confuses junior admins.

Let's say we're trying to find the switch port where a host at 10.77.77.10 is located, and the network's undocumented. The first step is fairly straightforward: traceroute.

```
R4#traceroute 10.77.77.10
Type escape sequence to abort.  
Tracing the route to 10.77.77.10
1 10.1.45.5 4 msec 0 msec 0 msec
2 10.1.56.6 4 msec 0 msec 4 msec
3 10.1.68.10 0 msec 4 msec 0 msec
4 10.77.77.10 4 msec *  0 msec
```

This tells us that the last router hop before the host is at 10.1.68.10. Assuming that we have access to that router, the next step is to examine its ARP table:

```
SW2#sh arp | i 10.77.77.10
Internet  10.77.77.10           189   000f.8f08.dad7  ARPA   Vlan77
```

In this case, the router's hostname gives us a clue that this device might be a layer 3 switch that terminates the VLAN in which the host resides. A couple of filtered commands can help us confirm this quickly:

```
SW2#sh ver | i IOS
Cisco IOS Software, C3560 Software (C3560-ADVIPSERVICESK9-M), Version 12.2(25)SEE, RELEASE SOFTWARE (fc2)

SW2#sh ip interface brief | i 77
Vlan77                 10.77.77.1      YES manual up                    up
```

The next step is to determine the interface on which the MAC address associated with the host's IP address resides:

```
SW2#sh mac-address-table | i dad7
  77    000f.8f08.dad7    DYNAMIC     Fa0/23
```

Next, we need to determine if there are any layer 2 switches connected to Fa0/23. The easiest way to do this is with CDP or LLDP. Obviously, it's possible that there could be a switch that's not running CDP or doesn't support it; in that case the biggest clue would be the number of MAC addresses located on Fa0/23. In this case, we'll just use CDP:

```
SW2#sh cdp nei fa0/23
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone
Device ID     Local Intrfce    Holdtme   Capability  Platform     Port ID
sw1.foo.com   Fas 0/23         170       S I         WS-C3560-2   Fas0/23
```

Next, we'll access sw1 via SSH from SW2. To find the IP address of sw1, we could use the conventional method of adding the "detail" keyword to the CDP command above. However, in the interest of exposing more useful and somewhat obscure IOS commands, we'll use a different command that produces less verbose output, giving us exactly what we want:

```
SW2#sh cdp entry sw1.foo.com protocol
Protocol information for sw1.foo.com :
  IP address: 10.77.77.100
```

Finally, we'll connect to sw1 and determine the interface on which our host resides:

```
SW2#ssh -l test 10.77.77.100 "sh mac-address-table | i dad7"
Password: <password entered>
  77    000f.8f08.dad7    DYNAMIC     Fa0/7
[Connection to 10.77.77.100 closed by foreign host]
```

Thus, we see that our host at 10.77.77.10 is located on Fa0/7 of sw1.foo.com. Note that with SSH, we didn't need to login interactively to execute a single command string. Instead, we just put the command string in quotes and executed as an argument to the "ssh" command.

In summary, to find the location of a host on the network:

1) Determine the last routed hop in the path to the host, preferably using traceroute.As an alternative to step 3, you could also use the layer 2 traceroute feature found in many Cisco switches. I've found this to be somewhat tedious, however, so I'll leave it as an exercise for the reader.

2) Connect to that router and search the ARP table for the host's IP address. Record the associated MAC address.

3) Search the MAC table for the host's MAC. Iterate this process through any remaining layer 2 switches.