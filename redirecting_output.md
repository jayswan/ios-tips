# Redirecting Output

## The Redirect Filter
Send the `show tech-support` output to a TFTP server:

```
Test#sh tech | redirect tftp://10.1.1.5/test-shTech.txt
!
Test#
```

### Appending Output to Concatenate a File in Flash

You can use the `append` filter to create or add to a file in flash. This is helpful when you need to save the output of a series of commands, but you can't or don't want to log your terminal session:

```
Test#sh version | append flash:output.txt

Test#sh run | append flash:output.txt

Test#more flash:output.txt
[output omitted]
Cisco 1841 (revision 6.0) with 238592K/23552K bytes of memory.
2 FastEthernet interfaces
1 Virtual Private Network (VPN) Module
DRAM configuration is 64 bits wide with parity disabled.
191K bytes of NVRAM.
62720K bytes of ATA CompactFlash (Read/Write)

Configuration register is 0x2102


Building configuration...

Current configuration : 1575 bytes
!
version 12.4
service timestamps debug datetime msec
service timestamps log datetime msec 
```
