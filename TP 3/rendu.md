# TP3 : On va router des trucs

## Sommaire

- [TP3 : On va router des trucs](#tp3--on-va-router-des-trucs)
  - [Sommaire](#sommaire)
  - [0. Prérequis](#0-prérequis)
  - [I. ARP](#i-arp)
  - [II. Routage](#ii-routage)

## I. [ARP](#i-arp)

## II. [Routage](#ii-routage)

# I. ARP

## 1. Echange ARP

🌞**Générer des requêtes ARP**
```
[et0@marcel ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=6.68 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=4.09 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=2.25 ms
64 bytes from 10.3.1.11: icmp_seq=4 ttl=64 time=2.08 ms
64 bytes from 10.3.1.11: icmp_seq=5 ttl=64 time=2.15 ms
^C
--- 10.3.1.11 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4013ms
rtt min/avg/max/mdev = 2.083/3.448/6.679/1.780 ms
```
```
[et0@john ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=3.92 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=2.06 ms
64 bytes from 10.3.1.12: icmp_seq=3 ttl=64 time=1.54 ms
64 bytes from 10.3.1.12: icmp_seq=4 ttl=64 time=7.03 ms
64 bytes from 10.3.1.12: icmp_seq=5 ttl=64 time=3.36 ms
^C
--- 10.3.1.12 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4015ms
rtt min/avg/max/mdev = 1.540/3.582/7.030/1.925 ms
```
```
[et0@marcel ~]$ ip neigh show
⭐ 10.3.1.11 ⭐ dev enp0s1 lladdr ⭐ 1e:73:7d:e2:69:19 ⭐ STALE 
```
```
[et0@john ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether ⭐ 1e:73:7d:e2:69:19 ⭐ brd ff:ff:ff:ff:ff:ff
    inet ⭐ 10.3.1.11/24 ⭐ brd 10.3.1.255 scope global noprefixroute enp0s1
       valid_lft forever preferred_lft forever
    inet6 fe80::1c73:7dff:fee2:6919/64 scope link 
       valid_lft forever preferred_lft forever
```

## 2. Analyse de trames

🌞**Analyse de trames**

**Voir tp3_arp.pcapng**

# II. Routage

Vous aurez besoin de 3 VMs pour cette partie. **Réutilisez les deux VMs précédentes.**

| Machine  | LAN 1 `10.3.1.0/24` | LAN 2 `10.3.2.0/24` |
| -------- | ------------------- | ------------------- |
| `router` | `10.3.1.254`        | `10.3.2.254`        |
| `john`   | `10.3.1.11`         | no                  |
| `marcel` | no                  | `10.3.2.12`         |

> Je les ai appelés `marcel` et `john` PASKON EN A MAR des noms nuls en réseau 🌻

```schema
   john                router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └───┘    └─────┘    └───┘    └─────┘
```

➜ **AVANT de continuer**, configurez correctement les IP sur les 3 VMs. Normalement :

```
[et0@john ~]$ ping 10.3.1.254
PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=4.01 ms
64 bytes from 10.3.1.254: icmp_seq=2 ttl=64 time=2.07 ms
64 bytes from 10.3.1.254: icmp_seq=3 ttl=64 time=5.29 ms
^C
--- 10.3.1.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2007ms
rtt min/avg/max/mdev = 2.066/3.789/5.292/1.326 ms
```
```
[et0@marcel ~]$ ping 10.3.2.254
PING 10.3.2.254 (10.3.2.254) 56(84) bytes of data.
64 bytes from 10.3.2.254: icmp_seq=1 ttl=64 time=2.35 ms
64 bytes from 10.3.2.254: icmp_seq=2 ttl=64 time=2.19 ms
64 bytes from 10.3.2.254: icmp_seq=3 ttl=64 time=2.10 ms
^C
--- 10.3.2.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2008ms
rtt min/avg/max/mdev = 2.097/2.213/2.350/0.104 ms
```

## 1. Mise en place du routage

➜ **Activer le routage sur le noeud `router`**

> Cette étape est nécessaire car Rocky Linux c'est pas un OS dédié au routage par défaut. Ce n'est bien évidemment une opération qui n'est pas nécessaire sur un équipement routeur dédié comme un routeur Cisco.

```bash
# pour autoriser une machine Linux à traiter des paquets IP qui ne lui sont pas destinés
# autrement dit "activer le routage"
$ sudo sysctl -w net.ipv4.ip_forward=1
```

🌞**Ajouter les routes statiques nécessaires pour que 
```
[et0@john ~]$ sudo cat /etc/sysconfig/network-scripts/route-enp0s1
[sudo] password for et0: 
⭐ 10.3.2.0/24 via 10.3.1.254 dev enp0s1 ⭐
```
```
[et0@marcel network-scripts]$ sudo cat /etc/sysconfig/network-scripts/route-enp0s1
⭐ 10.3.1.0/24 via 10.3.2.254 dev enp0s1 ⭐
```

## 2. Analyse de trames

🌞**Analyse des échanges ARP**
```
[et0@marcel ~]$ sudo ip neigh flush all
```
```
[et0@routeur ~]$ sudo ip neigh flush all
[et0@routeur ~]$ sudo tcpdump -i enp0s1 -c 10 -w tp3-arp-pinng-pong.pcapng not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```
```
[et0@john ~]$ sudo ip neigh flush all
[et0@john ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.94 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=1.09 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=3.72 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=3.30 ms
64 bytes from 10.3.2.12: icmp_seq=5 ttl=63 time=9.62 ms
^C
--- 10.3.2.12 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4017ms
rtt min/avg/max/mdev = 1.086/3.933/9.623/2.996 ms
```

| ordre | type trame  | IP source | MAC source                | IP destination | MAC destination            |
| ----- | ----------- | --------- | ------------------------- | -------------- | -------------------------- |
| 1     | Requête ARP | x         | `passerelle` `62:56:a0:06:3a:f4` | x              | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | x         | `marcel` `fa:86:63:92:96:7f`                         | x              | `passerelle` `62:56:a0:06:3a:f4`  |
| 3     | Ping        | `10.3.1.11`         | `62:56:a0:06:3a:f4`                         | `10.3.2.12`              | `fa:86:63:92:96:7f`                          |
| 4     | Pong        | `10.3.2.12`         |      `fa:86:63:92:96:7f`                    |    `10.3.1.11`           |      `62:56:a0:06:3a:f4`                   |

## 3. Accès internet

🌞**Donnez un accès internet à vos machines** - config routeur
```
[et0@routeur ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 5e:53:38:8e:15:c6 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.254/24 brd 10.3.1.255 scope global noprefixroute enp0s1
       valid_lft forever preferred_lft forever
    inet6 fe80::5c53:38ff:fe8e:15c6/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 06:df:d2:07:44:66 brd ff:ff:ff:ff:ff:ff
    inet 10.3.2.254/24 brd 10.3.2.255 scope global noprefixroute enp0s2
       valid_lft forever preferred_lft forever
    inet6 fe80::4df:d2ff:fe07:4466/64 scope link 
       valid_lft forever preferred_lft forever
4: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 42:b8:4a:80:0b:16 brd ff:ff:ff:ff:ff:ff
    inet ⭐ 10.3.3.2/24 ⭐ brd 10.3.3.255 scope global dynamic noprefixroute enp0s3
       valid_lft 84153sec preferred_lft 84153sec
    inet6 fd1c:f493:9200:a10:bcb2:b702:79a:a060/64 scope global dynamic noprefixroute 
       valid_lft 2591923sec preferred_lft 604723sec
    inet6 fe80::4043:578e:a83a:a541/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
```
[et0@routeur ~]$ ip r s
⭐ default via 10.3.3.1 dev enp0s3 proto dhcp src 10.3.3.2 metric 102 ⭐
10.3.1.0/24 dev enp0s1 proto kernel scope link src 10.3.1.254 metric 100 
10.3.2.0/24 dev enp0s2 proto kernel scope link src 10.3.2.254 metric 101 
10.3.3.0/24 dev enp0s3 proto kernel scope link src 10.3.3.2 metric 102 
```
```
[et0@routeur ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=56 time=23.4 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=56 time=26.3 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=56 time=73.2 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=56 time=142 ms
^C
--- 1.1.1.1 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 4003ms
rtt min/avg/max/mdev = 23.415/66.119/141.564/47.829 ms
```

🌞**Donnez un accès internet à vos machines** - config clients
```
[et0@john ~]$ ip r s
⭐ default via 10.3.1.254 dev enp0s1 ⭐
10.3.1.0/24 dev enp0s1 proto kernel scope link src 10.3.1.11 metric 100 
```
```
[et0@marcel ~]$ ip r s
⭐ default via 10.3.2.254 dev enp0s1 ⭐
10.3.2.0/24 dev enp0s1 proto kernel scope link src 10.3.2.12 metric 100 
```
```
[et0@john ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=55 time=31.7 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=55 time=23.7 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=55 time=36.7 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=55 time=32.4 ms
^C
--- 1.1.1.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3012ms
rtt min/avg/max/mdev = 23.734/31.132/36.650/4.669 ms
```
```
[et0@john ~]$ sudo cat /etc/resolv.conf 
# Generated by NetworkManager
search lan
⭐ nameserver 1.1.1.1 ⭐
nameserver fe80::bcd0:74ff:fe85:a864%enp0s1
```
```
[et0@marcel ~]$ sudo cat /etc/resolv.conf 
# Generated by NetworkManager
search lan
⭐ nameserver 1.1.1.1 ⭐
nameserver fe80::bcd0:74ff:fe85:a864%enp0s1
```
```
[et0@john ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=55 time=109 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=55 time=143 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=55 time=24.3 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=55 time=27.7 ms
^C
--- 1.1.1.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3010ms
rtt min/avg/max/mdev = 24.308/75.905/142.900/51.346 ms
```
```[et0@marcel ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=55 time=30.5 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=55 time=263 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=55 time=25.2 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=55 time=33.3 ms
^C
--- 1.1.1.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3009ms
rtt min/avg/max/mdev = 25.237/87.966/262.861/101.016 ms
```

```
[et0@marcel ~]$ curl apple.com
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
[et0@marcel ~]$ dig apple.com

; <<>> DiG 9.16.23-RH <<>> apple.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33724
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;apple.com.			IN	A

;; ANSWER SECTION:
apple.com.		305	IN	A	17.253.144.10

;; Query time: 60 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Thu Nov 16 14:28:23 CET 2023
;; MSG SIZE  rcvd: 54

[et0@marcel ~]$ ping apple.com
PING apple.com (17.253.144.10) 56(84) bytes of data.
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=1 ttl=52 time=58.8 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=2 ttl=52 time=46.2 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=3 ttl=52 time=93.2 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=4 ttl=52 time=58.5 ms
^C
--- apple.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3011ms
rtt min/avg/max/mdev = 46.167/64.164/93.197/17.519 ms
```
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
[et0@john ~]$ dig apple.com

; <<>> DiG 9.16.23-RH <<>> apple.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49937
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;apple.com.			IN	A

;; ANSWER SECTION:
apple.com.		855	IN	A	17.253.144.10

;; Query time: 70 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Thu Nov 16 14:31:20 CET 2023
;; MSG SIZE  rcvd: 54

[et0@john ~]$ ping apple.com
PING apple.com (17.253.144.10) 56(84) bytes of data.
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=1 ttl=52 time=51.3 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=2 ttl=52 time=45.4 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=3 ttl=52 time=83.6 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=4 ttl=52 time=48.6 ms
^C
--- apple.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 45.409/57.222/83.592/15.367 ms
```

🌞**Analyse de trames**

Voir : 🦈 **Capture réseau `tp3_routage_lan1.pcapng`**  
Voir : 🦈 **Capture réseau `tp3_routage_lan2.pcapng`**

- analysez un ping aller et le retour qui correspond et mettez dans un tableau :

| ordre | type trame | IP source            | MAC source                | IP destination | MAC destination |
| ----- | ---------- | -------------------- | ------------------------- | -------------- | --------------- |
| 1     | ping       | `john` `10.3.1.11` |  `de:04:f8:8c:0d:ac` | `routeur` `10.3.1.254`      | `5e:53:38:8e:15:c6`|
| 2     | ping       | `routeur` `10.3.2.254`                 | `06:df:d2:07:44:66`                       | `marcel` `10.3.2.12`            | `32:4e:83:c8:e5:ea`|
| 3     | pong       | `marcel` `10.3.2.12`                  |   `32:4e:83:c8:e5:ea`                     | `routeur` `10.3.2.254`            | `06:df:d2:07:44:66`          |
| 4     | pong       | `routeur` `10.3.1.254`| `5e:53:38:8e:15:c6`                       | `john` `10.3.1.11`           | `de:04:f8:8c:0d:ac` |

> Avec les deux captures vous devez observer 4 trames pour chaque ping échangé : 1) trame ping entre `john` et `router` (capture1) 2) trame ping entre `router` et `marcel` (capture2) 3) trame pong entre `marcel` et `router` (capture1) 4) trame pong entre `router` et `john` (capture2)

