# clab-crpdmpls
Juniper cRPD MPLS forwarding test with containerlab
![crpdmpls](https://user-images.githubusercontent.com/101124549/159582431-069c86e4-0d9a-4d35-a179-da74c8013875.png)


Juniper cRPD sample MPLS topology and configuration with containerlab

## Demo

### Pre-Requisite

+ Install [Docker](https://www.docker.com/)
+ Download, Load and Install [Juniper cRPD](https://www.juniper.net/gb/en/dm/crpd-free-trial.html)
  + Install Test License, else BGP wont work
+ Install [wbitt/network-multitool](https://hub.docker.com/r/wbitt/network-multitool) docker image
+ Install [containerlab](https://containerlab.dev/)
+ [Juniper cRPD Deployment Guide for Linux Server](https://www.juniper.net/documentation/us/en/software/crpd/crpd-deployment/topics/task/cRPD-Linux-Server-Docker-Routing-Mode.html) for reference

Note: I am running it in an ubuntu 18.04 LTS virtual machine

### Activate MPLS module in the host for MPLS forwarding between the containers

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
sysctl: reading key "net.ipv6.conf.all.stable_secret"
sysctl: reading key "net.ipv6.conf.default.stable_secret"
sysctl: reading key "net.ipv6.conf.docker0.stable_secret"
sysctl: reading key "net.ipv6.conf.ens3.stable_secret"
sysctl: reading key "net.ipv6.conf.erspan0.stable_secret"
sysctl: reading key "net.ipv6.conf.gretap0.stable_secret"
sysctl: reading key "net.ipv6.conf.ip6tnl0.stable_secret"
sysctl: reading key "net.ipv6.conf.lo.stable_secret"
sysctl: reading key "net.ipv6.conf.sit0.stable_secret"
net.mpls.conf.docker0.input = 0
net.mpls.conf.ens3.input = 0
net.mpls.conf.erspan0.input = 0
net.mpls.conf.gre0.input = 0
net.mpls.conf.gretap0.input = 0
net.mpls.conf.lo.input = 0
net.mpls.conf.sit0.input = 0
net.mpls.conf.tunl0.input = 0
net.mpls.default_ttl = 255
net.mpls.ip_ttl_propagate = 1
net.mpls.platform_labels = 1048575
lab@ubuntu1804:~$
```

### Clone the repository

