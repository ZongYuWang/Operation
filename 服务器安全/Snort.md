```py
[root@localhost snort-2.9.9.0]# /usr/local/snort/bin/snort 
```

*输出*
```py
WARNING: No preprocessors configured for policy 0.
08/18-22:51:44.161839 172.30.105.92:63671 -> 172.30.105.115:22
TCP TTL:128 TOS:0x0 ID:236 IpLen:20 DgmLen:92 DF
***AP*** Seq: 0xFD97A035  Ack: 0xBB46D677  Win: 0x4029  TcpLen: 20
=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+

^C*** Caught Int-Signal
08/18-22:51:44.162795 172.30.105.115:22 -> 172.30.105.92:63671
TCP TTL:64 TOS:0x10 ID:39577 IpLen:20 DgmLen:668 DF
***AP*** Seq: 0xBB46D7AB  Ack: 0xFD97A069  Win: 0x149  TcpLen: 20
=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+

===============================================================================
Run time for packet processing was 154.333411 seconds
Snort processed 356277 packets.
Snort ran for 0 days 0 hours 2 minutes 34 seconds
   Pkts/min:       178138
   Pkts/sec:         2313
===============================================================================
Memory usage summary:
  Total non-mmapped bytes (arena):       806912
  Bytes in mapped regions (hblkhd):      21590016
  Total allocated space (uordblks):      669856
  Total free space (fordblks):           137056
  Topmost releasable block (keepcost):   100032
===============================================================================
Packet I/O Totals:
   Received:       356278
   Analyzed:       356277 (100.000%)
    Dropped:            0 (  0.000%)
   Filtered:            0 (  0.000%)
Outstanding:            1 (  0.000%)
   Injected:            0
===============================================================================
Breakdown by protocol (includes rebuilt packets):
        Eth:       356277 (100.000%)
       VLAN:            0 (  0.000%)
        IP4:       356099 ( 99.950%)
       Frag:            0 (  0.000%)
       ICMP:          375 (  0.105%)
        UDP:          170 (  0.048%)
        TCP:       355488 ( 99.779%)
        IP6:           56 (  0.016%)
    IP6 Ext:           56 (  0.016%)
   IP6 Opts:            0 (  0.000%)
      Frag6:            0 (  0.000%)
      ICMP6:            0 (  0.000%)
       UDP6:           56 (  0.016%)
       TCP6:            0 (  0.000%)
     Teredo:            0 (  0.000%)
    ICMP-IP:            0 (  0.000%)
    IP4/IP4:            0 (  0.000%)
    IP4/IP6:            0 (  0.000%)
    IP6/IP4:            0 (  0.000%)
    IP6/IP6:            0 (  0.000%)
        GRE:            0 (  0.000%)
    GRE Eth:            0 (  0.000%)
   GRE VLAN:            0 (  0.000%)
    GRE IP4:            0 (  0.000%)
    GRE IP6:            0 (  0.000%)
GRE IP6 Ext:            0 (  0.000%)
   GRE PPTP:            0 (  0.000%)
    GRE ARP:            0 (  0.000%)
    GRE IPX:            0 (  0.000%)
   GRE Loop:            0 (  0.000%)
       MPLS:            0 (  0.000%)
        ARP:           27 (  0.008%)
        IPX:            0 (  0.000%)
   Eth Loop:           15 (  0.004%)
   Eth Disc:            0 (  0.000%)
   IP4 Disc:           63 (  0.018%)
   IP6 Disc:            0 (  0.000%)
   TCP Disc:            0 (  0.000%)
   UDP Disc:            0 (  0.000%)
  ICMP Disc:            0 (  0.000%)
All Discard:           63 (  0.018%)
      Other:           83 (  0.023%)
Bad Chk Sum:       252206 ( 70.789%)
    Bad TTL:            0 (  0.000%)
     S5 G 1:            0 (  0.000%)
     S5 G 2:            0 (  0.000%)
      Total:       356277
===============================================================================
Snort exiting

```