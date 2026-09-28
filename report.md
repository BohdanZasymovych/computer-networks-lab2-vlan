# Lab 2 - VLAN

## Initial GNS3 configuration

![Initial GNS3 configuration](./assets/initial_gns3_configuration.png)


## Nodes Setup

Interfaces on the nodes were named as follows: `ethN_XY`, where `N` is the interface number and `XY` is the name of the link between the two nodes that this interface connects to. `eth0` on each node was left untouched (used for management, configured via DHCP).

Static IP addresses were then assigned to each interface except `eth0`. To make them persistent, they were written to `/etc/network/interfaces`.

For instance, content of the `/etc/network/interfaces` for node-1:

```text
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

auto eth1_13
iface eth1_13 inet static
    address 10.0.13.1
    netmask 255.255.255.252

auto eth2_14
iface eth2_14 inet static
    address 10.0.14.1
    netmask 255.255.255.252
```

The 3rd octet of each address corresponds to the link between two nodes (`13`, `14`, `23`, `24`). The last octet identifies the node on that link: `1` for node-1/node-2, `2` for node-3/node-4.

| Node | Interface | IP address | Netmask | Connected to |
|------|-----------|------------|---------|--------------|
| node-1 | eth0 | DHCP | - | management |
| node-1 | eth1_13 | 10.0.13.1 | 255.255.255.252 | node-3 |
| node-1 | eth2_14 | 10.0.14.1 | 255.255.255.252 | node-4 |
| node-2 | eth0 | DHCP | - | management |
| node-2 | eth1_23 | 10.0.23.1 | 255.255.255.252 | node-3 |
| node-2 | eth2_24 | 10.0.24.1 | 255.255.255.252 | node-4 |
| node-3 | eth0 | DHCP | - | management |
| node-3 | eth1_13 | 10.0.13.2 | 255.255.255.252 | node-1 |
| node-3 | eth2_23 | 10.0.23.2 | 255.255.255.252 | node-2 |
| node-4 | eth0 | DHCP | - | management |
| node-4 | eth1_14 | 10.0.14.2 | 255.255.255.252 | node-1 |
| node-4 | eth2_24 | 10.0.24.2 | 255.255.255.252 | node-2 |


## VLAN Setup on the Switch

Add bridge:
```bash
root@switch:~$ ip l add name br1 type bridge
```

Enable `vlan_filtering`:
```bash
root@switch:~$ ip l set dev br1 type bridge vlan_filtering 1
```

Add all ports to the bridge and set them up:
```bash
root@switch:~$ for i in $(seq 0 12); do ip l set dev eth$i master br1 && ip l set eth$i up; done
```

Set bridge `br1` up:
```bash
root@switch:~$ ip l set br1 up
```

Add VLANs to which nodes are connected (via other interface than `eth0`):
```bash
root@switch:~$ bridge vlan add vid 13 dev eth1 pvid untagged
root@switch:~$ bridge vlan add vid 14 dev eth2 pvid untagged
root@switch:~$ bridge vlan add vid 23 dev eth4 pvid untagged
root@switch:~$ bridge vlan add vid 24 dev eth5 pvid untagged
root@switch:~$ bridge vlan add vid 13 dev eth7 pvid untagged
root@switch:~$ bridge vlan add vid 23 dev eth8 pvid untagged
root@switch:~$ bridge vlan add vid 14 dev eth10 pvid untagged
root@switch:~$ bridge vlan add vid 24 dev eth11 pvid untagged
```

Delete default `vid 1` from the ports above:
```bash
for i in 1 2 4 5 7 8 10 11; do bridge vlan del vid 1 dev eth$i; done
```

Verify result:
```bash
root@switch:~$ ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:00 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:01 brd ff:ff:ff:ff:ff:ff
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:02 brd ff:ff:ff:ff:ff:ff
5: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:03 brd ff:ff:ff:ff:ff:ff
6: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:04 brd ff:ff:ff:ff:ff:ff
7: eth5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:05 brd ff:ff:ff:ff:ff:ff
8: eth6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:06 brd ff:ff:ff:ff:ff:ff
9: eth7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:07 brd ff:ff:ff:ff:ff:ff
10: eth8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:08 brd ff:ff:ff:ff:ff:ff
11: eth9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:09 brd ff:ff:ff:ff:ff:ff
12: eth10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:0a brd ff:ff:ff:ff:ff:ff
13: eth11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:0b brd ff:ff:ff:ff:ff:ff
14: eth12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br1 state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:0c brd ff:ff:ff:ff:ff:ff
15: br1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:00 brd ff:ff:ff:ff:ff:ff

root@switch:~$ bridge vlan show
port              vlan-id  
eth0              1 PVID Egress Untagged
eth1              13 PVID Egress Untagged
eth2              14 PVID Egress Untagged
eth3              1 PVID Egress Untagged
eth4              23 PVID Egress Untagged
eth5              24 PVID Egress Untagged
eth6              1 PVID Egress Untagged
eth7              13 PVID Egress Untagged
eth8              23 PVID Egress Untagged
eth9              1 PVID Egress Untagged
eth10             14 PVID Egress Untagged
eth11             24 PVID Egress Untagged
eth12             1 PVID Egress Untagged
br1               1 PVID Egress Untagged
```

`ip l` confirms all ports (`eth0`-`eth12`) are enslaved to `br1` and up, and `bridge vlan show` confirms each node-facing port has the correct VLAN as its untagged PVID with the default `vid 1` removed.

To make these changes survive a reboot, the bridge setup was written to `/etc/network/interfaces`.

Content of the `/etc/network/interfaces`:
```text
auto lo
iface lo inet loopback

auto br1
iface br1 inet manual
    bridge_ports eth0 eth1 eth2 eth3 eth4 eth5 eth6 eth7 eth8 eth9 eth10 eth11 eth12
    bridge_vlan_filtering 1
    up bridge vlan add vid 13 dev eth1 pvid untagged
    up bridge vlan del vid 1 dev eth1
    up bridge vlan add vid 14 dev eth2 pvid untagged
    up bridge vlan del vid 1 dev eth2
    up bridge vlan add vid 23 dev eth4 pvid untagged
    up bridge vlan del vid 1 dev eth4
    up bridge vlan add vid 24 dev eth5 pvid untagged
    up bridge vlan del vid 1 dev eth5
    up bridge vlan add vid 13 dev eth7 pvid untagged
    up bridge vlan del vid 1 dev eth7
    up bridge vlan add vid 23 dev eth8 pvid untagged
    up bridge vlan del vid 1 dev eth8
    up bridge vlan add vid 14 dev eth10 pvid untagged
    up bridge vlan del vid 1 dev eth10
    up bridge vlan add vid 24 dev eth11 pvid untagged
    up bridge vlan del vid 1 dev eth11
```


## Connectivity Proof

To prove connectivity for each link between nodes, I ran `ping` on one node and `tcpdump` on the other, then repeated it in the opposite direction. If `ping` shows 0% packet loss and `tcpdump` shows the ICMP packets arriving, the link is healthy.

### Node 1 <-> Node 3 Connectivity

**Node-1 -> Node-3**

`ping` on the node-1:
```bash
root@node-1:~$ ping -c 5 -I eth1_13 10.0.13.2
PING 10.0.13.2 (10.0.13.2): 56 data bytes
64 bytes from 10.0.13.2: seq=0 ttl=64 time=1.755 ms
64 bytes from 10.0.13.2: seq=1 ttl=64 time=1.825 ms
64 bytes from 10.0.13.2: seq=2 ttl=64 time=1.868 ms
64 bytes from 10.0.13.2: seq=3 ttl=64 time=1.817 ms
64 bytes from 10.0.13.2: seq=4 ttl=64 time=1.916 ms

--- 10.0.13.2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.755/1.836/1.916 ms
```

`tcpdump` on the node-3:
```bash
root@node-3:~$ tcpdump -i eth1_13 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth1_13, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21:51:10.597411 IP 10.0.13.1 > 10.0.13.2: ICMP echo request, id 2507, seq 0, length 64
21:51:10.597482 IP 10.0.13.2 > 10.0.13.1: ICMP echo reply, id 2507, seq 0, length 64
21:51:11.597788 IP 10.0.13.1 > 10.0.13.2: ICMP echo request, id 2507, seq 1, length 64
21:51:11.597853 IP 10.0.13.2 > 10.0.13.1: ICMP echo reply, id 2507, seq 1, length 64
21:51:12.598189 IP 10.0.13.1 > 10.0.13.2: ICMP echo request, id 2507, seq 2, length 64
21:51:12.598255 IP 10.0.13.2 > 10.0.13.1: ICMP echo reply, id 2507, seq 2, length 64
21:51:13.598444 IP 10.0.13.1 > 10.0.13.2: ICMP echo request, id 2507, seq 3, length 64
21:51:13.598493 IP 10.0.13.2 > 10.0.13.1: ICMP echo reply, id 2507, seq 3, length 64
21:51:14.599236 IP 10.0.13.1 > 10.0.13.2: ICMP echo request, id 2507, seq 4, length 64
21:51:14.599305 IP 10.0.13.2 > 10.0.13.1: ICMP echo reply, id 2507, seq 4, length 64
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

**Node-3 -> Node-1**

`ping` on the node-3:
```bash
root@node-3:~$ ping -c 5 -I eth1_13 10.0.13.1
PING 10.0.13.1 (10.0.13.1): 56 data bytes
64 bytes from 10.0.13.1: seq=0 ttl=64 time=4.485 ms
64 bytes from 10.0.13.1: seq=1 ttl=64 time=2.046 ms
64 bytes from 10.0.13.1: seq=2 ttl=64 time=2.227 ms
64 bytes from 10.0.13.1: seq=3 ttl=64 time=4.124 ms
64 bytes from 10.0.13.1: seq=4 ttl=64 time=1.651 ms

--- 10.0.13.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.651/2.906/4.485 ms
```

`tcpdump` on the node-1:
```bash
root@node-1:~$ tcpdump -i eth1_13 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth1_13, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21:50:18.661519 IP 10.0.13.2 > 10.0.13.1: ICMP echo request, id 2517, seq 0, length 64
21:50:18.661587 IP 10.0.13.1 > 10.0.13.2: ICMP echo reply, id 2517, seq 0, length 64
21:50:19.659946 IP 10.0.13.2 > 10.0.13.1: ICMP echo request, id 2517, seq 1, length 64
21:50:19.660016 IP 10.0.13.1 > 10.0.13.2: ICMP echo reply, id 2517, seq 1, length 64
21:50:20.660481 IP 10.0.13.2 > 10.0.13.1: ICMP echo request, id 2517, seq 2, length 64
21:50:20.660550 IP 10.0.13.1 > 10.0.13.2: ICMP echo reply, id 2517, seq 2, length 64
21:50:21.660530 IP 10.0.13.2 > 10.0.13.1: ICMP echo request, id 2517, seq 3, length 64
21:50:21.660626 IP 10.0.13.1 > 10.0.13.2: ICMP echo reply, id 2517, seq 3, length 64
21:50:22.660941 IP 10.0.13.2 > 10.0.13.1: ICMP echo request, id 2517, seq 4, length 64
21:50:22.661013 IP 10.0.13.1 > 10.0.13.2: ICMP echo reply, id 2517, seq 4, length 64
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

### Node 1 <-> Node 4 Connectivity

**Node-1 -> Node-4**

`ping` on the node-1:
```bash
root@node-1:~$ ping -c 5 -I eth2_14 10.0.14.2
PING 10.0.14.2 (10.0.14.2): 56 data bytes
64 bytes from 10.0.14.2: seq=0 ttl=64 time=4.969 ms
64 bytes from 10.0.14.2: seq=1 ttl=64 time=1.954 ms
64 bytes from 10.0.14.2: seq=2 ttl=64 time=2.288 ms
64 bytes from 10.0.14.2: seq=3 ttl=64 time=1.756 ms
64 bytes from 10.0.14.2: seq=4 ttl=64 time=2.122 ms

--- 10.0.14.2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.756/2.617/4.969 ms
```

`tcpdump` on the node-4:
```bash
root@node-4:~$ tcpdump -i eth1_14 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth1_14, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21:54:48.239064 IP 10.0.14.1 > 10.0.14.2: ICMP echo request, id 2519, seq 0, length 64
21:54:48.239127 IP 10.0.14.2 > 10.0.14.1: ICMP echo reply, id 2519, seq 0, length 64
21:54:49.236362 IP 10.0.14.1 > 10.0.14.2: ICMP echo request, id 2519, seq 1, length 64
21:54:49.236455 IP 10.0.14.2 > 10.0.14.1: ICMP echo reply, id 2519, seq 1, length 64
21:54:50.237003 IP 10.0.14.1 > 10.0.14.2: ICMP echo request, id 2519, seq 2, length 64
21:54:50.237210 IP 10.0.14.2 > 10.0.14.1: ICMP echo reply, id 2519, seq 2, length 64
21:54:51.237232 IP 10.0.14.1 > 10.0.14.2: ICMP echo request, id 2519, seq 3, length 64
21:54:51.237313 IP 10.0.14.2 > 10.0.14.1: ICMP echo reply, id 2519, seq 3, length 64
21:54:52.237999 IP 10.0.14.1 > 10.0.14.2: ICMP echo request, id 2519, seq 4, length 64
21:54:52.238073 IP 10.0.14.2 > 10.0.14.1: ICMP echo reply, id 2519, seq 4, length 64
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

**Node-4 -> Node-1**

`ping` on the node-4:
```bash
root@node-4:~$ ping -c 5 -I eth1_14 10.0.14.1
PING 10.0.14.1 (10.0.14.1): 56 data bytes
64 bytes from 10.0.14.1: seq=0 ttl=64 time=1.964 ms
64 bytes from 10.0.14.1: seq=1 ttl=64 time=1.985 ms
64 bytes from 10.0.14.1: seq=2 ttl=64 time=2.436 ms
64 bytes from 10.0.14.1: seq=3 ttl=64 time=2.440 ms
64 bytes from 10.0.14.1: seq=4 ttl=64 time=2.035 ms

--- 10.0.14.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.964/2.172/2.440 ms
```

`tcpdump` on the node-1:
```bash
root@node-1:~$ tcpdump -i eth2_14 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth2_14, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21:59:02.724018 IP 10.0.14.2 > 10.0.14.1: ICMP echo request, id 2523, seq 0, length 64
21:59:02.724104 IP 10.0.14.1 > 10.0.14.2: ICMP echo reply, id 2523, seq 0, length 64
21:59:03.724711 IP 10.0.14.2 > 10.0.14.1: ICMP echo request, id 2523, seq 1, length 64
21:59:03.724742 IP 10.0.14.1 > 10.0.14.2: ICMP echo reply, id 2523, seq 1, length 64
21:59:04.725453 IP 10.0.14.2 > 10.0.14.1: ICMP echo request, id 2523, seq 2, length 64
21:59:04.725517 IP 10.0.14.1 > 10.0.14.2: ICMP echo reply, id 2523, seq 2, length 64
21:59:05.726052 IP 10.0.14.2 > 10.0.14.1: ICMP echo request, id 2523, seq 3, length 64
21:59:05.726131 IP 10.0.14.1 > 10.0.14.2: ICMP echo reply, id 2523, seq 3, length 64
21:59:06.726444 IP 10.0.14.2 > 10.0.14.1: ICMP echo request, id 2523, seq 4, length 64
21:59:06.726496 IP 10.0.14.1 > 10.0.14.2: ICMP echo reply, id 2523, seq 4, length 64
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

### Node 2 <-> Node 3 Connectivity

**Node-2 -> Node-3**

`ping` on the node-2:
```bash
root@node-2:~$ ping -c 5 -I eth1_23 10.0.23.2
PING 10.0.23.2 (10.0.23.2): 56 data bytes
64 bytes from 10.0.23.2: seq=0 ttl=64 time=1.886 ms
64 bytes from 10.0.23.2: seq=1 ttl=64 time=1.840 ms
64 bytes from 10.0.23.2: seq=2 ttl=64 time=2.354 ms
64 bytes from 10.0.23.2: seq=3 ttl=64 time=2.436 ms
64 bytes from 10.0.23.2: seq=4 ttl=64 time=2.124 ms

--- 10.0.23.2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.840/2.128/2.436 ms
```

`tcpdump` on the node-3:
```bash
root@node-3:~$ tcpdump -i eth2_23 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth2_23, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21:51:58.196844 IP 10.0.23.1 > 10.0.23.2: ICMP echo request, id 2515, seq 0, length 64
21:51:58.196922 IP 10.0.23.2 > 10.0.23.1: ICMP echo reply, id 2515, seq 0, length 64
21:51:59.197397 IP 10.0.23.1 > 10.0.23.2: ICMP echo request, id 2515, seq 1, length 64
21:51:59.197467 IP 10.0.23.2 > 10.0.23.1: ICMP echo reply, id 2515, seq 1, length 64
21:52:00.198139 IP 10.0.23.1 > 10.0.23.2: ICMP echo request, id 2515, seq 2, length 64
21:52:00.198213 IP 10.0.23.2 > 10.0.23.1: ICMP echo reply, id 2515, seq 2, length 64
21:52:01.199007 IP 10.0.23.1 > 10.0.23.2: ICMP echo request, id 2515, seq 3, length 64
21:52:01.199133 IP 10.0.23.2 > 10.0.23.1: ICMP echo reply, id 2515, seq 3, length 64
21:52:02.199477 IP 10.0.23.1 > 10.0.23.2: ICMP echo request, id 2515, seq 4, length 64
21:52:02.199505 IP 10.0.23.2 > 10.0.23.1: ICMP echo reply, id 2515, seq 4, length 64
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

**Node-3 -> Node-2**

`ping` on the node-3:
```bash
root@node-3:~$ ping -c 5 -I eth2_23 10.0.23.1
PING 10.0.23.1 (10.0.23.1): 56 data bytes
64 bytes from 10.0.23.1: seq=0 ttl=64 time=1.561 ms
64 bytes from 10.0.23.1: seq=1 ttl=64 time=2.084 ms
64 bytes from 10.0.23.1: seq=2 ttl=64 time=1.958 ms
64 bytes from 10.0.23.1: seq=3 ttl=64 time=2.304 ms
64 bytes from 10.0.23.1: seq=4 ttl=64 time=2.091 ms

--- 10.0.23.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.561/1.999/2.304 ms
```

`tcpdump` on the node-2:
```bash
root@node-2:~$ tcpdump -i eth1_23 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth1_23, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21:48:04.666633 IP 10.0.23.2 > 10.0.23.1: ICMP echo request, id 2516, seq 0, length 64
21:48:04.666689 IP 10.0.23.1 > 10.0.23.2: ICMP echo reply, id 2516, seq 0, length 64
21:48:05.667287 IP 10.0.23.2 > 10.0.23.1: ICMP echo request, id 2516, seq 1, length 64
21:48:05.667360 IP 10.0.23.1 > 10.0.23.2: ICMP echo reply, id 2516, seq 1, length 64
21:48:06.667768 IP 10.0.23.2 > 10.0.23.1: ICMP echo request, id 2516, seq 2, length 64
21:48:06.667834 IP 10.0.23.1 > 10.0.23.2: ICMP echo reply, id 2516, seq 2, length 64
21:48:07.668449 IP 10.0.23.2 > 10.0.23.1: ICMP echo request, id 2516, seq 3, length 64
21:48:07.668562 IP 10.0.23.1 > 10.0.23.2: ICMP echo reply, id 2516, seq 3, length 64
21:48:08.669363 IP 10.0.23.2 > 10.0.23.1: ICMP echo request, id 2516, seq 4, length 64
21:48:08.669453 IP 10.0.23.1 > 10.0.23.2: ICMP echo reply, id 2516, seq 4, length 64
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

### Node 2 <-> Node 4 Connectivity

**Node-2 -> Node-4**

`ping` on the node-2:
```bash
root@node-2:~$ ping -c 5 -I eth2_24 10.0.24.2
PING 10.0.24.2 (10.0.24.2): 56 data bytes
64 bytes from 10.0.24.2: seq=0 ttl=64 time=7.144 ms
64 bytes from 10.0.24.2: seq=1 ttl=64 time=1.608 ms
64 bytes from 10.0.24.2: seq=2 ttl=64 time=1.867 ms
64 bytes from 10.0.24.2: seq=3 ttl=64 time=1.668 ms
64 bytes from 10.0.24.2: seq=4 ttl=64 time=1.957 ms

--- 10.0.24.2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.608/2.848/7.144 ms
```

`tcpdump` on the node-4:
```bash
root@node-4:~$ tcpdump -i eth2_24 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth2_24, link-type EN10MB (Ethernet), snapshot length 262144 bytes
22:01:29.623613 IP 10.0.24.1 > 10.0.24.2: ICMP echo request, id 2528, seq 0, length 64
22:01:29.623676 IP 10.0.24.2 > 10.0.24.1: ICMP echo reply, id 2528, seq 0, length 64
22:01:30.618818 IP 10.0.24.1 > 10.0.24.2: ICMP echo request, id 2528, seq 1, length 64
22:01:30.618890 IP 10.0.24.2 > 10.0.24.1: ICMP echo reply, id 2528, seq 1, length 64
22:01:31.619397 IP 10.0.24.1 > 10.0.24.2: ICMP echo request, id 2528, seq 2, length 64
22:01:31.619487 IP 10.0.24.2 > 10.0.24.1: ICMP echo reply, id 2528, seq 2, length 64
22:01:32.619751 IP 10.0.24.1 > 10.0.24.2: ICMP echo request, id 2528, seq 3, length 64
22:01:32.619800 IP 10.0.24.2 > 10.0.24.1: ICMP echo reply, id 2528, seq 3, length 64
22:01:33.620247 IP 10.0.24.1 > 10.0.24.2: ICMP echo request, id 2528, seq 4, length 64
22:01:33.620335 IP 10.0.24.2 > 10.0.24.1: ICMP echo reply, id 2528, seq 4, length 64
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

**Node-4 -> Node-2**

`ping` on the node-4:
```bash
root@node-4:~$ ping -c 5 -I eth2_24 10.0.24.1
PING 10.0.24.1 (10.0.24.1): 56 data bytes
64 bytes from 10.0.24.1: seq=0 ttl=64 time=1.532 ms
64 bytes from 10.0.24.1: seq=1 ttl=64 time=2.379 ms
64 bytes from 10.0.24.1: seq=2 ttl=64 time=1.883 ms
64 bytes from 10.0.24.1: seq=3 ttl=64 time=1.657 ms
64 bytes from 10.0.24.1: seq=4 ttl=64 time=1.951 ms

--- 10.0.24.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.532/1.880/2.379 ms
```

`tcpdump` on the node-2:
```bash
root@node-2:~$ tcpdump -i eth2_24 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth2_24, link-type EN10MB (Ethernet), snapshot length 262144 bytes
22:02:10.497004 IP 10.0.24.2 > 10.0.24.1: ICMP echo request, id 2527, seq 0, length 64
22:02:10.497095 IP 10.0.24.1 > 10.0.24.2: ICMP echo reply, id 2527, seq 0, length 64
22:02:11.497810 IP 10.0.24.2 > 10.0.24.1: ICMP echo request, id 2527, seq 1, length 64
22:02:11.497986 IP 10.0.24.1 > 10.0.24.2: ICMP echo reply, id 2527, seq 1, length 64
22:02:12.498227 IP 10.0.24.2 > 10.0.24.1: ICMP echo request, id 2527, seq 2, length 64
22:02:12.498298 IP 10.0.24.1 > 10.0.24.2: ICMP echo reply, id 2527, seq 2, length 64
22:02:13.498527 IP 10.0.24.2 > 10.0.24.1: ICMP echo request, id 2527, seq 3, length 64
22:02:13.498597 IP 10.0.24.1 > 10.0.24.2: ICMP echo reply, id 2527, seq 3, length 64
22:02:14.499009 IP 10.0.24.2 > 10.0.24.1: ICMP echo request, id 2527, seq 4, length 64
22:02:14.499086 IP 10.0.24.1 > 10.0.24.2: ICMP echo reply, id 2527, seq 4, length 64
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```
