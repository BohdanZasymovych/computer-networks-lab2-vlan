# Lab 2 - VLAN

## Initial GNS3 configuration

![Initial GNS3 configuration](./assets/initial_gns3_configuration.png)


## Nodes Setup

Interfaces on the nodes were named in a next way: `ethN_XY`, where `N` is the number of the interface and `XY` name of the link between two nodes this interface is used. `eth0` for each is left untouched.

Then static ip addresses where created for each interface except `eth0`. To make them persistent they were written to `/etc/network/interfaces`

For instance content of the `/etc/network/interfaces` for node-1:

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

3-rd byte in the addresses correspond to the link between two nodes; last byte identifies machine on the link `1` for node-1 and node-2, `2` for node-3 and node-4.


## VLAN Setup on the Switch

Add bridge:
```bash
root@switch:~$ ip l add name br1 type bridge
root@switch:~$ ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 0c:cb:45:60:00:00 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:01 brd ff:ff:ff:ff:ff:ff
4: eth2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:02 brd ff:ff:ff:ff:ff:ff
5: eth3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:03 brd ff:ff:ff:ff:ff:ff
6: eth4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:04 brd ff:ff:ff:ff:ff:ff
7: eth5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:05 brd ff:ff:ff:ff:ff:ff
8: eth6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:06 brd ff:ff:ff:ff:ff:ff
9: eth7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:07 brd ff:ff:ff:ff:ff:ff
10: eth8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:08 brd ff:ff:ff:ff:ff:ff
11: eth9: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:09 brd ff:ff:ff:ff:ff:ff
12: eth10: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:0a brd ff:ff:ff:ff:ff:ff
13: eth11: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:0b brd ff:ff:ff:ff:ff:ff
14: eth12: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 0c:cb:45:60:00:0c brd ff:ff:ff:ff:ff:ff
15: br1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether a6:84:21:68:7f:b6 brd ff:ff:ff:ff:ff:ff
```

Enable `vlan_filtering`:
```bash
root@switch:~$ ip l set dev br1 type bridge vlan_filtering 1
root@switch:~$ ip -d link show br1
15: br1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether a6:84:21:68:7f:b6 brd ff:ff:ff:ff:ff:ff promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535 
    bridge forward_delay 1500 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 1 vlan_protocol 802.1Q bridge_id 8000.00:00:00:00:00:00 designated_root 8000.00:00:00:00:00:00 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer    0.00 fdb_n_learned 0 fdb_max_learned 0 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 no_linklocal_learn 0 mcast_vlan_snooping 0 mst_enabled 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536 
```

Add all ports to the bridge and set them up

```bash
root@switch:~$ for i in $(seq 0 12); do ip l set dev eth$i master br1 && ip l set eth$i up; done
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
15: br1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 0c:cb:45:60:00:00 brd ff:ff:ff:ff:ff:ff
```

Set bridge `br1` up:

```bash
root@switch:~$ ip l set br1 up
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

To make this changes survive reboot setup for the bridge was written in the `/etc/network/interfaces`

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