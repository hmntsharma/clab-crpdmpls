# clab-crpdmpls
Juniper cRPD MPLS forwarding test with containerlab
![crpdmpls](https://user-images.githubusercontent.com/101124549/159582431-069c86e4-0d9a-4d35-a179-da74c8013875.png)


Juniper cRPD sample MPLS topology and configuration with containerlab

## Demo

### Pre-Requisite

+ Install [Docker](https://www.docker.com/)
+ Download, Load and Install [Juniper cRPD](https://www.juniper.net/gb/en/dm/crpd-free-trial.html)
  + Install test license, else BGP won't work
+ Install [wbitt/network-multitool](https://hub.docker.com/r/wbitt/network-multitool) docker image
+ Install [containerlab](https://containerlab.dev/)
+ [Juniper cRPD Deployment Guide for Linux Server](https://www.juniper.net/documentation/us/en/software/crpd/crpd-deployment/topics/task/cRPD-Linux-Server-Docker-Routing-Mode.html) for reference

Note: I am running it in an ubuntu 18.04 LTS virtual machine, 8vCPU and 16GB RAM


### Activate MPLS module and Increase label limit in the host for MPLS forwarding between the containers

```
lab@ubuntu1804:~$ sudo modprobe mpls_iptunnel
lab@ubuntu1804:~$ sudo modprobe mpls_router
lab@ubuntu1804:~$ sudo modprobe ip_tunnel
lab@ubuntu1804:~$ sudo sysctl -w net.mpls.platform_labels=1048575
net.mpls.platform_labels = 1048575
lab@ubuntu1804:~$ 
```

Verify
```
lab@ubuntu1804:~$ lsmod | grep mpls
mpls_iptunnel          16384  0
mpls_router            28672  1 mpls_iptunnel
ip_tunnel              24576  4 ipip,ip_gre,sit,mpls_router
lab@ubuntu1804:~$ 

lab@ubuntu1804:~$ sudo sysctl -a | grep mpls
<..snipped..>
net.mpls.platform_labels = 1048575
lab@ubuntu1804:~$
```

### Clone the repository
```
lab@ubuntu1804:~/clab$ sudo git clone https://github.com/w1nt3rfell/clab-crpdmpls.git
Cloning into 'clab-crpdmpls'...
remote: Enumerating objects: 110, done.
remote: Counting objects: 100% (110/110), done.
remote: Compressing objects: 100% (78/78), done.
remote: Total 110 (delta 37), reused 74 (delta 25), pack-reused 0
Receiving objects: 100% (110/110), 55.79 KiB | 1.74 MiB/s, done.
Resolving deltas: 100% (37/37), done.
lab@ubuntu1804:~/
```

### Deploy the lab

```
lab@ubuntu1804:~/github/clab-crpdmpls$ sudo clab deploy -t crpdmpls.yml
INFO[0000] Containerlab v0.25.1 started
INFO[0000] Parsing & checking topology file: crpdmpls.yml
INFO[0000] Creating lab directory: /home/lab/github/clab-crpdmpls/clab-crpdmpls
INFO[0000] Creating docker network: Name="clab", IPv4Subnet="172.20.20.0/24", IPv6Subnet="2001:172:20:20::/64", MTU="1500"
INFO[0000] config file '/home/lab/github/clab-crpdmpls/clab-crpdmpls/PE1/config/juniper.conf' for node 'PE1' already exists and will not be generated/reset
INFO[0000] Creating container: "HOST1"
INFO[0000] Creating container: "PE1"
INFO[0000] Creating container: "HOST3"
INFO[0000] config file '/home/lab/github/clab-crpdmpls/clab-crpdmpls/PE3/config/juniper.conf' for node 'PE3' already exists and will not be generated/reset
INFO[0000] config file '/home/lab/github/clab-crpdmpls/clab-crpdmpls/CR2/config/juniper.conf' for node 'CR2' already exists and will not be generated/reset
INFO[0000] Creating container: "PE3"
INFO[0000] Creating container: "CR2"
INFO[0003] Creating virtual wire: PE3:eth3 <--> HOST3:eth3
INFO[0004] Creating virtual wire: CR2:eth2 <--> PE3:eth2
INFO[0005] Creating virtual wire: PE1:eth1 <--> CR2:eth1
INFO[0005] Creating virtual wire: PE1:eth3 <--> HOST1:eth3
INFO[0005] Adding containerlab host entries to /etc/hosts file
+---+-------+--------------+-------------------------+-------+---------+----------------+----------------------+
| # | Name  | Container ID |          Image          | Kind  |  State  |  IPv4 Address  |     IPv6 Address     |
+---+-------+--------------+-------------------------+-------+---------+----------------+----------------------+
| 1 | CR2   | 876f05b7ac26 | crpd:21.4R1.12          | crpd  | running | 172.20.20.5/24 | 2001:172:20:20::5/64 |
| 2 | HOST1 | 5f15f832ab43 | wbitt/network-multitool | linux | running | 172.20.20.2/24 | 2001:172:20:20::2/64 |
| 3 | HOST3 | 631e71cb49d8 | wbitt/network-multitool | linux | running | 172.20.20.3/24 | 2001:172:20:20::3/64 |
| 4 | PE1   | ab2639c318e5 | crpd:21.4R1.12          | crpd  | running | 172.20.20.4/24 | 2001:172:20:20::4/64 |
| 5 | PE3   | 864f65e23ec0 | crpd:21.4R1.12          | crpd  | running | 172.20.20.6/24 | 2001:172:20:20::6/64 |
+---+-------+--------------+-------------------------+-------+---------+----------------+----------------------+
lab@ubuntu1804:~/github/clab-crpdmpls$
```

### Verify Linux interface configuration

cRPD uses Linux kernel forwarding, for which all the interface config is done in the shell.

The "linux_net_config" directory contains the necessary configuration for all the routers and hosts.

The 'exec' command in the .yml file configures it on the nodes, once the lab is deployed.

```
root@PE1:/# ip addr show eth1
238: eth1@if237: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9500 qdisc noqueue state UP group default
    link/ether aa:c1:ab:ab:a1:80 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 10.1.2.1/24 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a8c1:abff:feab:a180/64 scope link
       valid_lft forever preferred_lft forever
root@PE1:/# ip addr show eth3
240: eth3@if239: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9500 qdisc noqueue master __crpd-vrf1 state UP group default
    link/ether aa:c1:ab:41:48:1d brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet 192.168.100.2/24 scope global eth3
       valid_lft forever preferred_lft forever
    inet6 fe80::a8c1:abff:fe41:481d/64 scope link
       valid_lft forever preferred_lft forever
root@PE1:/# ip addr show lo
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 1.1.1.1/32 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
root@PE1:/#
```

```
lab@ubuntu1804:~$ sudo docker exec -it HOST1 bash
bash-5.1# ip addr show eth3
239: eth3@if240: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9500 qdisc noqueue state UP group default
    link/ether aa:c1:ab:3c:71:e1 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 192.168.100.1/24 scope global eth3
       valid_lft forever preferred_lft forever
    inet6 fe80::a8c1:abff:fe3c:71e1/64 scope link
       valid_lft forever preferred_lft forever
bash-5.1# ip route show
default via 192.168.100.2 dev eth3
172.20.20.0/24 dev eth0 proto kernel scope link src 172.20.20.2
192.168.100.0/24 dev eth3 proto kernel scope link src 192.168.100.1
bash-5.1#
```

### Useful Info

>cRPD needs both the loopback interfaces in the ISIS.
>```
>root@PE1> show configuration | display set | match "isis.*lo"
>set protocols isis interface lo.0
>set protocols isis interface lo0.0
>root@PE1>
>```

### Verify Dynamic Routing and Forwarding 
```
root@PE1> show isis interface
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
eth1                  2   0x1 Disabled          Point to Point         10/100
lo.0                  2   0x1 Passive           Passive                 0/0

root@PE1> show isis adjacency
Interface             System         L State        Hold (secs) SNPA
eth1                  CR2            2  Up                   24

root@PE1> show isis database level 2 detail
IS-IS level 2 link-state database:

PE1.00-00 Sequence: 0x3, Checksum: 0xb1d8, Lifetime: 1091 secs
   IS neighbor: CR2.00                        Metric:      100
   IP prefix: 1.1.1.1/32                      Metric:        0 Internal Up
   IP prefix: 10.1.2.0/24                     Metric:      100 Internal Up

CR2.00-00 Sequence: 0x4, Checksum: 0x9d82, Lifetime: 1024 secs
   IS neighbor: PE1.00                        Metric:      100
   IS neighbor: PE3.00                        Metric:      100
   IP prefix: 2.2.2.2/32                      Metric:        0 Internal Up
   IP prefix: 10.1.2.0/24                     Metric:      100 Internal Up
   IP prefix: 10.2.3.0/24                     Metric:      100 Internal Up

PE3.00-00 Sequence: 0x3, Checksum: 0xd395, Lifetime: 1052 secs
   IS neighbor: CR2.00                        Metric:      100
   IP prefix: 3.3.3.3/32                      Metric:        0 Internal Up
   IP prefix: 10.2.3.0/24                     Metric:      100 Internal Up

root@PE1> show ldp route fec-only detail
Destination                            Next-hop intf/lsp/table  Next-hop address
 1.1.1.1/32                            lo.0
   Bound to outgoing label 3, Topology entry: 0x55bdf1b05780
   Ingress route status: Inactive
   Last event(s): Evaluate Update history
   Route type: Egress route
Destination                            Next-hop intf/lsp/table  Next-hop address
 2.2.2.2/32                            eth1                     10.1.2.2
   Session ID 1.1.1.1:0--2.2.2.2:0
   Bound to outgoing label 17, Topology entry: 0x55bdf1267380
   Ingress route status: Active, Last modified: 00:16:07 ago
   Last event(s): Evaluate Delete and add transit route
   Route flags: Ingress TTL propagate, Transit TTL propagate
Destination                            Next-hop intf/lsp/table  Next-hop address
 3.3.3.3/32                            eth1                     10.1.2.2
   Session ID 1.1.1.1:0--2.2.2.2:0
   Bound to outgoing label 18, Topology entry: 0x55bdf1b08f00
   Ingress route status: Active, Last modified: 00:15:58 ago
   Last event(s): Evaluate Delete and add transit route
   Route flags: Ingress TTL propagate, Transit TTL propagate

root@PE1> show bgp summary
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 1 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.l3vpn.0
                       1          1          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
3.3.3.3               65001        109        108       0       0       15:45 Establ
  bgp.l3vpn.0: 1/1/1/0
  CRPD.inet.0: 1/1/1/0

root@PE1> show route

inet.0: 9 destinations, 9 routes (9 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

1.1.1.1/32         *[Direct/0] 00:17:00
                    >  via lo.0
2.2.2.2/32         *[IS-IS/18] 00:16:14, metric 100
                    >  to 10.1.2.2 via eth1
3.3.3.3/32         *[IS-IS/18] 00:16:04, metric 200
                    >  to 10.1.2.2 via eth1
10.1.2.0/24        *[Direct/0] 00:17:00
                    >  via eth1
10.1.2.1/32        *[Local/0] 00:17:00
                       Local via eth1
10.2.3.0/24        *[IS-IS/18] 00:16:14, metric 200
                    >  to 10.1.2.2 via eth1
172.20.20.0/24     *[Direct/0] 00:17:00
                    >  via eth0
172.20.20.4/32     *[Local/0] 00:17:00
                       Local via eth0
224.0.0.2/32       *[LDP/9] 00:17:00, metric 1
                       MultiRecv

inet.3: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2.2.2.2/32         *[LDP/9] 00:16:13, metric 1
                    >  to 10.1.2.2 via eth1
3.3.3.3/32         *[LDP/9] 00:16:04, metric 1
                    >  to 10.1.2.2 via eth1, Push 16

CRPD.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.100.0/24   *[Direct/0] 00:17:00
                    >  via eth3
192.168.100.2/32   *[Local/0] 00:17:00
                       Local via eth3
192.168.200.0/24   *[BGP/170] 00:15:47, localpref 100, from 3.3.3.3
                      AS path: I, validation-state: unverified
                    >  to 10.1.2.2 via eth1, Push 16, Push 16(top)

iso.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

49.0001.0000.0000.0001/72
                   *[Direct/0] 00:17:00
                    >  via lo.0

mpls.0: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 00:17:00, metric 1
                       Receive
1                  *[MPLS/0] 00:17:00, metric 1
                       Receive
2                  *[MPLS/0] 00:17:00, metric 1
                       Receive
13                 *[MPLS/0] 00:17:00, metric 1
                       Receive
16                 *[VPN/0] 00:17:00
                    >  via __crpd-vrf1 (CRPD), Pop
17                 *[LDP/9] 00:16:13, metric 1
                    >  to 10.1.2.2 via eth1, Pop
17(S=0)            *[LDP/9] 00:16:13, metric 1
                    >  to 10.1.2.2 via eth1, Pop
18                 *[LDP/9] 00:16:04, metric 1
                    >  to 10.1.2.2 via eth1, Swap 16

bgp.l3vpn.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

65001:100:192.168.200.0/24
                   *[BGP/170] 00:15:47, localpref 100, from 3.3.3.3
                      AS path: I, validation-state: unverified
                    >  to 10.1.2.2 via eth1, Push 16, Push 16(top)

inet6.0: 15 destinations, 19 routes (15 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

::/96              *[Direct/0] 00:17:00
                    >  via sit0
                    [Direct/0] 00:17:00
                    >  via sit0
                    [Direct/0] 00:17:00
                    >  via sit0
                    [Direct/0] 00:17:00
                    >  via sit0
                    [Direct/0] 00:17:00
                    >  via sit0
::1.1.1.1/128      *[Local/0] 00:17:00
                       Local via sit0
::10.1.2.1/128     *[Local/0] 00:17:00
                       Local via sit0
::127.0.0.1/128    *[Local/0] 00:17:00
                       Local via sit0
::172.20.20.4/128  *[Local/0] 00:17:00
                       Local via sit0
::192.168.100.2/128*[Local/0] 00:17:00
                       Local via sit0
2001:172:20:20::/64*[Direct/0] 00:17:00
                    >  via eth0
2001:172:20:20::4/128
                   *[Local/0] 00:17:00
                       Local via eth0
fe80::1/128        *[Direct/0] 00:17:00
                    >  via lo.0
fe80::26:2dff:fedc:fe05/128
                   *[Local/0] 00:16:58
                       Local via irb
fe80::42:acff:fe14:1404/128
                   *[Local/0] 00:17:00
                       Local via eth0
fe80::308c:66ff:fed7:e907/128
                   *[Local/0] 00:17:00
                       Local via ip6tnl0
fe80::a80b:ebff:fe57:e711/128
                   *[Local/0] 00:17:00
                       Local via lsi
fe80::a8c1:abff:feab:a180/128
                   *[Local/0] 00:17:00
                       Local via eth1
ff02::2/128        *[INET6/0] 00:17:00
                       MultiRecv

CRPD.inet6.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

fe80::a8c1:abff:fe41:481d/128
                   *[Local/0] 00:17:00
                       Local via eth3
ff02::2/128        *[INET6/0] 00:17:00
                       MultiRecv

root@PE1>
```

```
root@PE1:/# ip route show
default via 172.20.20.1 dev eth0
2.2.2.2 via 10.1.2.2 dev eth1 proto 22
3.3.3.3 via 10.1.2.2 dev eth1 proto 22
10.1.2.0/24 dev eth1 proto kernel scope link src 10.1.2.1
10.2.3.0/24 via 10.1.2.2 dev eth1 proto 22
172.20.20.0/24 dev eth0 proto kernel scope link src 172.20.20.4
root@PE1:/#
root@PE1:/#
root@PE1:/# ip -f mpls route show
16 dev __crpd-vrf1 proto 22
17 via inet 10.1.2.2 dev eth1 proto 22
18 as to 16 via inet 10.1.2.2 dev eth1 proto 22
root@PE1:/#
root@PE1:/#
root@PE1:/# ping -c2 3.3.3.3 -I 1.1.1.1
PING 3.3.3.3 (3.3.3.3) from 1.1.1.1 : 56(84) bytes of data.
64 bytes from 3.3.3.3: icmp_seq=1 ttl=63 time=0.298 ms
64 bytes from 3.3.3.3: icmp_seq=2 ttl=63 time=0.098 ms

--- 3.3.3.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1016ms
rtt min/avg/max/mdev = 0.098/0.198/0.298/0.100 ms
root@PE1:/#
```


### End to End communication via MPLS forwarding

#### Ping from HOST1 to HOST3
```
lab@ubuntu1804:~$ sudo docker exec HOST1 ping -c1 192.168.200.1
PING 192.168.200.1 (192.168.200.1) 56(84) bytes of data.
64 bytes from 192.168.200.1: icmp_seq=1 ttl=62 time=0.125 ms

--- 192.168.200.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.125/0.125/0.125/0.000 ms
lab@ubuntu1804:~$ 
```

#### tcpdump on PE3 eth2 interface: verify labeled traffic
```
root@PE3:/# tcpdump -i eth2 mpls
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth2, link-type EN10MB (Ethernet), capture size 262144 bytes
22:07:07.850814 MPLS (label 16, exp 0, [S], ttl 63) IP 192.168.100.1 > 192.168.200.1: ICMP echo request, id 33, seq 1, length 64
22:07:07.850875 MPLS (label 17, exp 0, ttl 63) (label 16, exp 0, [S], ttl 63) IP 192.168.200.1 > 192.168.100.1: ICMP echo reply, id 33, seq 1, length 64
```

#### tcpdump on HOST3 eth3 interface
```
lab@ubuntu1804:~$ sudo docker exec -it HOST3 bash
bash-5.1# tcpdump -i eth3 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth3, link-type EN10MB (Ethernet), snapshot length 262144 bytes
22:10:27.551420 IP 192.168.100.1 > 192.168.200.1: ICMP echo request, id 37, seq 1, length 64
22:10:27.551437 IP 192.168.200.1 > 192.168.100.1: ICMP echo reply, id 37, seq 1, length 64
```

**Thank You!**
