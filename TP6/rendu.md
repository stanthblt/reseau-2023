# TP6 : Un LAN maîtrisé meo ?

![Not the network](./img/not_the_network.jpg)

## Sommaire

- [TP6 : Un LAN maîtrisé meo ?](#tp6--un-lan-maîtrisé-meo-)
  - [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
- [I. Setup routeur](#i-setup-routeur)
- [II. Serveur DNS](#ii-serveur-dns)
  - [1. Présentation](#1-présentation)
  - [2. Setup](#2-setup)
  - [3. Test](#3-test)
- [III. Serveur DHCP](#iii-serveur-dhcp)
- [IV. Bonus : Serveur Web HTTPS](#iv-bonus--serveur-web-https)

# 0. Prérequis
 
➜ **L'emoji 🖥️ indique une VM à créer**. Pour chaque VM, vous déroulerez la checklist suivante :

- [x] Dès que le serveur DNS est en place, n'oubliez pas de l'ajouter à la conf des autres machines à la main si nécessaire

# I. Setup routeur

| Name            | LAN1 `10.6.1.0/24` |
| --------------- | ------------------ |
| `router.tp6.b1` | `10.6.1.254`       |
| `john.tp6.b1`   | `10.6.1.11`        |

🖥️ **Machine `router.tp6.b1`**

```
$ sudo firewall-cmd --add-masquerade --permanent
$ sudo firewall-cmd --reload
```

🖥️ **Machine `john.tp6.b1`**


# II. Serveur DNS


## 1. Présentation

## 2. Setup


| Name            | LAN1 `10.6.1.0/24` |
| --------------- | ------------------ |
| `router.tp6.b1` | `10.6.1.254`       |
| `john.tp6.b1`   | `10.6.1.11`        |
| `dns.tp6.b1`    | `10.6.1.101`       |

🖥️ **Machine `dns.tp6.b1`**

- n'oubliez pas de dérouler la checklist

Installation du serveur DNS :

```bash
# installation du serveur DNS, son p'tit nom c'est BIND9
$ sudo dnf install -y bind bind-utils
```

La configuration du serveur DNS va se faire dans 3 fichiers essentiellement :

- **un fichier de configuration principal**
  - `/etc/named.conf`
  - on définit les trucs généraux, comme les adresses IP et le port où on veu écouter
  - on définit aussi un chemin vers les autres fichiers, les fichiers de zone
- **un fichier de zone**
  - `/var/named/tp6.b1.db`
  - je vous préviens, la syntaxe fait mal
  - on peut y définir des correspondances `nom ---> IP`
- **un fichier de zone inverse**
  - `/var/named/tp6.b1.rev`
  - on peut y définir des correspondances `IP ---> nom`

➜ **Allooooons-y, fichier de conf principal**

- je ne vous mets que les lignes les plus importantes, et celles qu'on modifie
- les `[...]` indiquent qu'il faut laisser la conf par défaut, que j'ai enlevé ici pour que ce soit + lisible
  - il ne faut donc pas les mettre dans vos fichiers

```bash
# éditez le fichier de config principal pour qu'il ressemble à :
$ sudo cat /etc/named.conf
options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
[...]
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };

        recursion yes;
[...]
# référence vers notre fichier de zone
zone "tp6.b1" IN {
     type master;
     file "tp6.b1.db";
     allow-update { none; };
     allow-query {any; };
};
# référence vers notre fichier de zone inverse
zone "1.4.10.in-addr.arpa" IN {
     type master;
     file "tp6.b1.rev";
     allow-update { none; };
     allow-query { any; };
};
```

➜ **Et pour les fichiers de zone**

- dans ces fichiers c'est le caractère `;` pour les commentaires
- hésitez pas à virer mes commentaires de façon générale
- c'juste pour que vous captiez mais ça fait dégueu dans un fichier de conf

```bash
# Fichier de zone pour nom -> IP

$ sudo cat /var/named/tp6.b1.db

$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns       IN A 10.6.1.101
john      IN A 10.6.1.11
```

```bash
# Fichier de zone inverse pour IP -> nom

$ sudo cat /var/named/tp6.b1.rev

$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Reverse lookup
101 IN PTR dns.tp6.b1.
11 IN PTR john.tp6.b1.
```

➜ **Une fois ces 3 fichiers en place, démarrez le service DNS**

```bash
# Démarrez le service tout de suite
$ sudo systemctl start named

# Faire en sorte que le service démarre tout seul quand la VM s'allume
$ sudo systemctl enable named

# Obtenir des infos sur le service
$ sudo systemctl status named

# Obtenir des logs en cas de probème
$ sudo journalctl -xe -u named
```

🌞 **Dans le rendu, je veux**

`Fichier principal :`
```
[et0@dns ~]$ sudo cat /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file	"/var/named/data/named.secroots";
	recursing-file	"/var/named/data/named.recursing";
	allow-query     { localhost; any; };
	allow-query-cache { localhost; any; };

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-validation yes;

	managed-keys-directory "/var/named/dynamic";
	geoip-directory "/usr/share/GeoIP";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";

	/* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
	include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

zone "tp6.b1" IN {
     type master;
     file "tp6.b1.db";
     allow-update { none; };
     allow-query {any; };
};

zone "1.4.10.in-addr.arpa" IN {
     type master;
     file "tp6.b1.rev";
     allow-update { none; };
     allow-query { any; };
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```
`Premier fichier zone :`
```
[et0@dns ~]$ sudo cat /var/named/tp6.b1.db
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns       IN A 10.6.1.101
john      IN A 10.6.1.11
```
`Deuxième fichier zone :`
```
[et0@dns ~]$ sudo cat /var/named/tp6.b1.rev
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Reverse lookup
101 IN PTR dns.tp6.b1.
11 IN PTR john.tp6.b1.
```
`Status named :`
```
[et0@dns ~]$ sudo systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: disabled)
     Active: active (running) since Fri 2023-11-24 15:15:32 CET; 4min 21s ago
   Main PID: 11879 (named)
      Tasks: 8 (limit: 4263)
     Memory: 24.4M
        CPU: 52ms
     CGroup: /system.slice/named.service
             └─11879 /usr/sbin/named -u named -c /etc/named.conf

Nov 24 15:15:32 dns.tp6.b1 named[11879]: network unreachable resolving './DNSKEY/IN': 2001:dc3::35#53
Nov 24 15:15:32 dns.tp6.b1 named[11879]: network unreachable resolving './NS/IN': 2001:dc3::35#53
Nov 24 15:15:32 dns.tp6.b1 named[11879]: zone 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip>
Nov 24 15:15:32 dns.tp6.b1 named[11879]: network unreachable resolving './DNSKEY/IN': 2001:500:2f::f#53
Nov 24 15:15:32 dns.tp6.b1 named[11879]: network unreachable resolving './NS/IN': 2001:500:2f::f#53
Nov 24 15:15:32 dns.tp6.b1 named[11879]: all zones loaded
Nov 24 15:15:32 dns.tp6.b1 named[11879]: running
Nov 24 15:15:32 dns.tp6.b1 systemd[1]: Started Berkeley Internet Name Domain (DNS).
Nov 24 15:15:32 dns.tp6.b1 named[11879]: managed-keys-zone: Initializing automatic trust anchor management for z>
Nov 24 15:15:32 dns.tp6.b1 named[11879]: resolver priming query complete
lines 1-20/20 (END)
```
`ss :`
```
[et0@dns ~]$ sudo ss -lntpu
udp            UNCONN           0                0                             10.6.1.101:53                            0.0.0.0:*              users:(("named",pid=11879,fd=31))
tcp            LISTEN           0                10                            10.6.1.101:53                            0.0.0.0:*              users:(("named",pid=11879,fd=32))
```

🌞 **Ouvrez le bon port dans le firewall**

```
[et0@dns ~]$ sudo firewall-cmd --add-port=53/udp --permanent
success
```
```
[et0@dns ~]$ sudo firewall-cmd --reload && sudo firewall-cmd --list-all
success
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s1
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 53/udp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

## 3. Test

🌞 **Sur la machine `john.tp6.b1`**
`config carte enp0s1`
```
[et0@john ~]$ sudo cat /etc/sysconfig/network-scripts/ifcfg-enp0s1
DEVICE=enp0s1

BOOTPROTO=static
ONOOT=yes

IPADDR=10.6.1.11
NETMASK=255.255.255.0

DNS1=10.6.1.101
```
`config DNS`
```
[et0@john ~]$ sudo cat /etc/resolv.conf
# Generated by NetworkManager
search tp6.b1
nameserver 10.6.1.101
```
`Ping dns.tp6.b1`
```
[et0@john ~]$ ping dns.tp6.b1
PING dns.tp6.b1 (10.6.1.101) 56(84) bytes of data.
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=1 ttl=64 time=0.814 ms
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=2 ttl=64 time=3.93 ms
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=3 ttl=64 time=2.21 ms
^C
--- dns.tp6.b1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.814/2.319/3.932/1.275 ms
```
`Ping www.ynov.com`
```
[et0@john ~]$ ping www.ynov.com
PING www.ynov.com (172.67.74.226) 56(84) bytes of data.
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=1 ttl=51 time=40.5 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=2 ttl=51 time=64.3 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=3 ttl=51 time=62.0 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=4 ttl=51 time=106 ms
^C
--- www.ynov.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 4170ms
rtt min/avg/max/mdev = 40.519/68.124/105.620/23.555 ms
```

🌞 **Sur votre PC**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % nslookup john.tp6.b1
Server:		10.6.1.101
Address:	10.6.1.101#53

Non-authoritative answer:
Name:	john.tp6.b1
Address: 10.6.1.11
```
- utilisez une commande pour résoudre le nom `john.tp6.b1` en utilisant `10.6.1.101` comme serveur DNS

🦈 **Capture `tp6_dns.pcapng`**

- une requête DNS vers le nom `john.tp6.b1` ainsi que la réponse
- prenez deux minutes pour regarder le contenu de la trame
- les requêtes DNS passent en clair, rien n'est chiffré : on voit tout !
- capture à réaliser avec une commande `tcpdump` sur l'un des VMs

# III. Serveur DHCP


| Name            | LAN1 `10.6.1.0/24` |
| --------------- | ------------------ |
| `router.tp6.b1` | `10.6.1.254`       |
| `john.tp6.b1`   | `DHCP`             |
| `dns.tp6.b1`    | `10.6.1.101`       |
| `dhcp.tp6.b1`   | `10.6.1.102`       |

🖥️ **Machine `dhcp.tp6.b1`**

- déroulez la checklisteuh
- indiquez votre propre serveur DNS pour la résolution de noms

🌞 **Installer un serveur DHCP**

```
[et0@dhcp ~]$ sudo dnf -y install dhcp-server
Last metadata expiration check: 1:32:21 ago on Fri Nov 24 15:44:38 2023.
Package dhcp-server-12:4.4.2-19.b1.el9.aarch64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```
```
[et0@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
#
# create new
# specify domain name
option domain-name     "srv.world";
# specify DNS server's hostname or IP address
option domain-name-servers     10.6.1.101;
# default lease time
default-lease-time 21600;
# max lease time
max-lease-time 25200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.6.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.6.1.13 10.6.1.37;
    # specify broadcast address
    option broadcast-address 10.6.1.255;
    # specify gateway
    option routers 10.6.1.254;
}
```
```
[et0@dhcp ~]$ sudo systemctl enable --now dhcpd
```
```
[et0@dhcp ~]$ sudo firewall-cmd --add-service=dhcp
success
```
```
[et0@dhcp ~]$ sudo firewall-cmd --runtime-to-permanent
success
```

🌞 **Test avec `john.tp6.b1`**

- enlevez-lui son IP statique, et récupérez une IP en DHCP
```
[et0@john ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 3e:f2:cc:eb:2d:c3 brd ff:ff:ff:ff:ff:ff
    inet 10.6.1.14/24 brd 10.6.1.255 scope global dynamic enp0s1
       valid_lft 84716sec preferred_lft 84716sec
    inet6 fe80::3cf2:ccff:feeb:2dc3/64 scope link 
       valid_lft forever preferred_lft forever
```
```
[et0@john ~]$ ip r s
default via 10.6.1.254 dev enp0s1 proto dhcp src 10.6.1.14 metric 100
10.6.1.0/24 dev enp0s1 proto kernel scope link src 10.6.1.14 metric 100
```
```
[et0@john ~]$ sudo cat /etc/resolv.conf
# Generated by NetworkManager
search srv.world
nameserver 10.6.1.101
```
```
[et0@john ~]$ ping google.com
PING google.com (142.250.75.238) 56(84) bytes of data.
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=1 ttl=115 time=12.7 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=2 ttl=115 time=13.9 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 12.725/13.306/13.887/0.581 ms
```

🌞 **Requête web avec `john.tp6.b1`**

```
[et0@john ~]$ curl apple.com
<HTML>
<HEAD>
<TITLE>Document Has Moved</TITLE>
</HEAD>

<BODY BGCOLOR="white" FGCOLOR="black">
<H1>Document Has Moved</H1>
<HR>

<FONT FACE="Helvetica,Arial"><B>
Description: The document you requested has moved to a new location.  The new location is "https://www.apple.com/".
</B></FONT>
<HR>
</BODY>
```

🦈 **Capture `tp6_web.pcapng`**