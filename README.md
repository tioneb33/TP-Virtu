# TP1 : Remise dans le bain


# I. Most simplest LAN

🌞 **Déterminer l'adresse MAC de vos deux machines**

~~~
 show ip
~~~
🌞 **Définir une IP statique sur les deux machines**

~~~
ip (adresse ip)
save
~~~

Machines:
node1.tp1.efrei / IP: 10.1.1.1/24 / MAC: 00:50:79:66:68:00
node2.tp1.efrei / IP: 10.1.1.2/24 / MAC: 00:50:79:66:68:01


🌞 **Effectuer un `ping` d'une machine à l'autre**

~~~
VPCS> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=1.288 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=0.327 ms
~~~



🌞 **Wireshark !**

Protocole: ICMP

[ICMP](./wireshark.ping.etape1.pcapng)

🌞 **ARP**

~~~
VPCS> show arp

00:50:79:66:68:01  10.1.1.2 expires in 108 seconds
~~~

# II. Ajoutons un switch


🌞 **Déterminer l'adresse MAC de vos trois machines**

node1.tp1.efrei / IP: 10.1.1.1/24 / MAC: 00:50:79:66:68:00
node1.tp2.efrei / IP: 10.1.1.2/24 / MAC: 00:50:79:66:68:00
node1.tp3.efrei / IP: 10.1.1.3/24 / MAC: 00:50:79:66:68:00

🌞 **Définir une IP statique sur les trois machines**

~~~
ip (adresse ip)
save
show ip
~~~

🌞 **Effectuer des `ping` d'une machine à l'autre**

~~~
node1 à node2 : 

VPCS> ping 10.1.1.2
84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=7.582 ms

node2 à node3 :

VPCS> ping 10.1.1.3
84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=5.799 ms

node1 à node3 : 

VPCS> ping 10.1.1.3
84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=5.106 ms
~~~


# III. Serveur DHCP

## 1. Legit

🌞 **Donner un accès Internet à la machine `dhcp.tp1.efrei`**

Pour vérifier : ping 8.8.8.8
et après install de dhcp

🌞 **Installer et configurer un serveur DHCP**

~~~
dnf-y install dhcp-server
vi /etc/dhcp/dhcpd.conf
systemctl enable --now dhcpd

authoritative;
subnet 10.1.1.0 netmask 255.255.255.0 {
	range 10.1.1.10 10.1.1.50;
	option broadcast-address 10.1.1.1;
	option routers 10.1.1.1;
}
~~~

🌞 **Récupérer une IP automatiquement depuis les 3 nodes**

~~~
PC1> dhcp
DORA IP 10.1.1.10/24 GW 10.1.1.1

PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.1.1.10/24
GATEWAY     : 10.1.1.1
DNS         : 
DHCP SERVER : 10.1.1.253
DHCP LEASE  : 563, 570/285/498
MAC         : 00:50:79:66:68:01
LPORT       : 20007
RHOST:PORT  : 127.0.0.1:20008
MTU         : 1500

PC2> dhcp
DDORA IP 10.1.1.12/24 GW 10.1.1.1

PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.1.1.12/24
GATEWAY     : 10.1.1.1
DNS         : 
DHCP SERVER : 10.1.1.253
DHCP LEASE  : 594, 599/299/524
MAC         : 00:50:79:66:68:00
LPORT       : 20009
RHOST:PORT  : 127.0.0.1:20010
MTU         : 1500

PC3> dhcp
DDORA IP 10.1.1.11/24 GW 10.1.1.1

PC3> show ip

NAME        : PC3[1]
IP/MASK     : 10.1.1.11/24
GATEWAY     : 10.1.1.1
DNS         : 
DHCP SERVER : 10.1.1.253
DHCP LEASE  : 584, 599/299/524
MAC         : 00:50:79:66:68:02
LPORT       : 20011
RHOST:PORT  : 127.0.0.1:20012
MTU         : 1500
~~~


🌞 **Wireshark !**

[DHCP](./wireshark.ping.etape3.pcapng)

## 2. DHCP spoofing

🌞 **Configurez dnsmasq**

~~~
dnf install -y dnsmasq

port=0 # pour désactiver DNS
dhcp-range=10.1.1.210,10.1.1.250,255.255.255.0,12h
dhcp-authoritative
interface=enp0s3
~~~

🌞 **Test !**

~~~
PC4> dhcp
DDORA IP 10.1.1.248/24 GW 10.1.1.50
PC4> show ip

NAME        : PC4[1]
IP/MASK     : 10.1.1.248/24
~~~

🌞 **Now race !**

~~~
PC2> dhcp
DDORA IP 10.1.1.246/24 GW 10.1.1.50
PC3> dhcp
DDORA IP 10.1.1.247/24 GW 10.1.1.50
PC4> dhcp
DORA IP 10.1.1.21/24 GW 10.1.1.1
~~~

🌞 **Wireshark !**

[race](./wireshark.ping.etape3.2pcapng.pcapng)