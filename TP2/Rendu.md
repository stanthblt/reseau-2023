# TP2 : Ethernet, IP, et ARP

Dans ce TP on va approfondir trois protocoles, qu'on a survolé jusqu'alors :

- **IPv4** *(Internet Protocol Version 4)* : gestion des adresses IP
  - on va aussi parler d'ICMP, de DHCP, bref de tous les potes d'IP quoi !
- **Ethernet** : gestion des adresses MAC
- **ARP** *(Address Resolution Protocol)* : permet de trouver l'adresse MAC de quelqu'un sur notre réseau dont on connaît l'adresse IP

![Seventh Day](./img/tcpip.jpg)

# Sommaire

- [TP2 : Ethernet, IP, et ARP](#tp2--ethernet-ip-et-arp)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
- [I. Setup IP](#i-setup-ip)
- [II. ARP my bro](#ii-arp-my-bro)
- [III. DHCP](#iii-dhcp)

# 0. Prérequis

# I. Setup IP

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % sipcalc 10.10.10.0/30
-[ipv4 : 10.10.10.0/30] - 0

[CIDR]
Host address		- ⭐ 10.10.10.0 ⭐
Host address (decimal)	- 168430080
Host address (hex)	- A0A0A00
Network address		- 10.10.10.0
Network mask		- ⭐ 255.255.255.252 ⭐
Network mask (bits)	- 30
Network mask (hex)	- FFFFFFFC
Broadcast address	- ⭐ 10.10.10.3 ⭐
Cisco wildcard		- 0.0.0.3
Addresses in network	- 4
Network range		- 10.10.10.0 - 10.10.10.3
Usable range		- ⭐ 10.10.10.1 ⭐ - ⭐ 10.10.10.2 ⭐
```

🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % ping 10.10.10.2
PING 10.10.10.2 (10.10.10.2): 56 data bytes
64 bytes from 10.10.10.2: icmp_seq=0 ttl=128 time=1.259 ms
64 bytes from 10.10.10.2: icmp_seq=1 ttl=128 time=1.611 ms
64 bytes from 10.10.10.2: icmp_seq=2 ttl=128 time=1.805 ms
64 bytes from 10.10.10.2: icmp_seq=3 ttl=128 time=1.340 ms
64 bytes from 10.10.10.2: icmp_seq=4 ttl=128 time=1.336 ms
^C
--- 10.10.10.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 1.259/1.470/1.805/0.206 ms
```

🌞 **Wireshark it**

**Voir rendu.pcapng**

# II. ARP my bro

🌞 **Check the ARP table**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % arp -a 
? (10.33.64.86) at 32:a2:82:ce:6:25 on en0 ifscope [ethernet]
? (10.33.64.126) at 62:48:42:7f:26:d0 on en0 ifscope [ethernet]
? (10.10.10.2) at ⭐ 2c:f0:5d:6c:11:b8 ⭐ on en0 ifscope [ethernet]
? (10.33.65.14) at 50:5a:65:6:29:13 on en0 ifscope [ethernet]
? (10.33.65.98) at a6:76:f0:23:26:d1 on en0 ifscope [ethernet]
? (10.33.65.99) at 56:cc:12:43:16:b4 on en0 ifscope [ethernet]
? (10.33.65.102) at f0:18:98:b5:76:c5 on en0 ifscope 
[ethernet]
? (10.33.79.254) at ⭐ 7c:5a:1c:d3:d8:76 ⭐ on en0 ifscope [ethernet]
```

🌞 **Manipuler la table ARP**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % sudo arp -ad
10.33.64.86 (10.33.64.86) deleted
10.33.64.121 (10.33.64.121) deleted
10.33.64.126 (10.33.64.126) deleted
10.33.64.209 (10.33.64.209) deleted
10.33.64.211 (10.33.64.211) deleted
10.33.65.14 (10.33.65.14) deleted
10.33.65.92 (10.33.65.92) deleted
10.33.65.98 (10.33.65.98) deleted
10.33.65.99 (10.33.65.99) deleted
10.33.65.102 (10.33.65.102) deleted
10.33.65.103 (10.33.65.103) deleted
10.33.65.110 (10.33.65.110) deleted
10.33.65.113 (10.33.65.113) deleted
```
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % arp -a      
? (10.33.65.93) at 5c:e9:1e:91:95:9b on en0 ifscope [ethernet]
? (10.33.65.104) at d0:c6:37:79:3d:b7 on en0 ifscope [ethernet]
? (10.33.65.201) at b2:fc:17:6a:49:ec on en0 ifscope 
[ethernet]
? (10.10.10.2) at ⭐ 2c:f0:5d:6c:11:b8 ⭐ on en0 ifscope [ethernet]
? (10.33.66.24) at fe:84:e1:d5:2a:b8 on en0 ifscope [ethernet]
? (10.33.66.25) at 60:6e:e8:b7:8b:66 on en0 ifscope [ethernet]
? (10.33.66.119) at 50:e0:85:b7:db:a8 on en0 ifscope [ethernet]
```

🌞 **Wireshark it**

- vous savez maintenant comment forcer un échange ARP : il sufit de vider la table ARP et tenter de contacter quelqu'un, l'échange ARP se fait automatiquement
- mettez en évidence les deux trames ARP échangées lorsque vous essayez de contacter quelqu'un pour la "première" fois
  - déterminez, pour les deux trames, les adresses source et destination
  - déterminez à quoi correspond chacune de ces adresses

🦈 **PCAP qui contient les DEUX trames ARP**

> L'échange ARP est constitué de deux trames : un ARP broadcast et un ARP reply.

# III. DHCP


*DHCP* pour *Dynamic Host Configuration Protocol* est notre p'tit pote qui nous file des IPs quand on arrive dans un réseau, parce que c'est chiant de le faire à la main :)

Quand on arrive dans un réseau, notre PC contacte un serveur DHCP, et récupère généralement 3 infos :

- **1.** une IP à utiliser
- **2.** l'adresse IP de la passerelle du réseau
- **3.** l'adresse d'un serveur DNS joignable depuis ce réseau

L'échange DHCP  entre un client et le serveur DHCP consiste en 4 trames : **DORA**, que je vous laisse chercher sur le web vous-mêmes : D

🌞 **Wireshark it**

- identifiez les 4 trames DHCP lors d'un échange DHCP
  - mettez en évidence les adresses source et destination de chaque trame
- identifiez dans ces 4 trames les informations **1**, **2** et **3** dont on a parlé juste au dessus

🦈 **PCAP qui contient l'échange DORA**

> **Soucis** : l'échange DHCP ne se produit qu'à la première connexion. **Pour forcer un échange DHCP**, ça dépend de votre OS. Sur **GNU/Linux**, avec `dhclient` ça se fait bien. Sur **Windows**, le plus simple reste de définir une IP statique pourrie sur la carte réseau, se déconnecter du réseau, remettre en DHCP, se reconnecter au réseau (ché moa sa march). Essayez de regarder par vous-mêmes s'il existe une commande clean pour faire ça ! Sur **MacOS**, je connais peu mais Internet dit qu'c'est po si compliqué, appelez moi si besoin.