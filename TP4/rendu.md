# TP4 : DHCP

Dans ce TP on va jouer avec **le protocole DHCP**.

D'abord du point de vue d'un **client**, puis vous allez vous-mêmes **monter un serveur DHCP**.

- [TP4 : DHCP](#tp4--dhcp)
- [I. DHCP Client](#i-dhcp-client)
  - [II. Serveur DHCP](#ii-serveur-dhcp)
    - [1. Topologie](#1-topologie)
    - [2. Tableau d'adressage](#2-tableau-dadressage)
    - [3. Setup topologie](#3-setup-topologie)
    - [4. Serveur DHCP](#4-serveur-dhcp)
    - [5. Client DHCP](#5-client-dhcp)
    - [6. Options DHCP](#6-options-dhcp)

# I. DHCP Client

Pour cette partie, vous utiliserez uniquement votre PC (pas de VM), et la ligne de commande.

🌞 **Déterminer**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % ipconfig getpacket en0                       
op = BOOTREPLY
htype = 1
flags = 0
hlen = 6
hops = 0
xid = 0x153f00f9
secs = 0
ciaddr = 0.0.0.0
yiaddr = 10.33.70.193
siaddr = 0.0.0.0
giaddr = 0.0.0.0
chaddr = bc:d0:74:58:6f:5e
sname = 
file = 
options:
Options count is 7
dhcp_message_type (uint8): ACK 0x5
server_identifier (ip): ⭐ 10.33.79.254 ⭐
lease_time (uint32): 0x1517a
subnet_mask (ip): 255.255.240.0
router (ip_mult): {10.33.79.254}
domain_name_server (ip_mult): {8.8.8.8, 1.1.1.1}
end (none): 
```
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % sudo cat /var/db/dhcpclient/leases/en0.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>ClientIdentifier</key>
	<data>
	AbzQdFhvXg==
	</data>
	<key>IPAddress</key>
	<string>10.33.70.193</string>
	⭐ <key>LeaseLength</key>
	<integer>86394</integer> ⭐
	⭐ <key>LeaseStartDate</key>
	<date>2023-10-27T06:57:49Z</date> ⭐
	<key>NetworkID</key>
	<string>88EEAB2A-43CE-41E6-98A0-63B8DE094A7B</string>
	<key>PacketData</key>
	<data>
	AgEGABU/APkAAAAAAAAAAAohRsEAAAAAAAAAALzQdFhvXgAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABjglNjNQEFNgQKIU/+MwQAAVF6
	AQT///AAAwQKIU/+BggICAgIAQEBAf8AAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	</data>
	<key>RouterHardwareAddress</key>
	<data>
	fFoc09h2
	</data>
	<key>RouterIPAddress</key>
	<string>10.33.79.254</string>
	<key>SSID</key>
	<string>WiFi@YNOV</string>
</dict>
</plist>
```
- l'adresse du serveur DHCP : 
    ```10.33.79.254```
- l'heure exacte à laquelle vous avez obtenu votre bail DHCP : ```vendredi 27 octobre 2023 08:57:49 GMT+02:00 DST```
- l'heure exacte à laquelle il va expirer : ```samedi 28 octobre 2023 08:57:43 GMT+02:00 DST```

🌞 **Capturer un échange DHCP**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % sudo ifconfig en0 down
```
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % sudo ipconfig set en0 DHCP
```
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % sudo ifconfig en0 up
```

🌞 **Analyser la capture Wireshark**

- parmi ces 4 trames, ```DHCP Offer``` contient les informations proposées au client
- en cliquant sur l'une des 4 trames, et en dépliant la partie DHCP (en bas dans l'interface de Wireshark) vous pourrez repérer ces informations
    - ```Your (client) IP address: 10.33.70.193```
    - ```Option 3 : Routeur = 10.33.79.254```
    - ```Option 6 : DNS = 8.8.8.8; 1.1.1.1```

🦈 **`tp4_dhcp_client.pcapng`**

## II. Serveur DHCP

Pour cette partie, on sort les VMs ! On va monter un petit LAN au sein duquel un serveur DHCP pourra fournir des adresses IP aux clients, ainsi que d'autres informations en plus d'une IP.

Respectez scrupuleusement la checklist suivante sur chaque VM, avant de vous attaquer à la suite :

- [ ] IP statique (sauf mention contraire)
- [ ] Connexion SSH fonctionnelle
- [ ] Vous avez défini un nom à la machine (voir [mémo](../../cours/memo/rocky_network.md))
- [ ] Pas de carte NAT (sauf mention contraire)
- [ ] Accès internet
  - [ ] ajout d'une route par défaut si nécessaire
  - [ ] ajout de l'adresse d'un serveur DNS si nécessaire

### 1. Topologie

```schema
                node2.tp4.b1
                  ┌──────┐
                  │      │
                  │      │
                  │      │
                  └──┬───┘
                     │
                     │
  ┌──────┐           │           ┌──────┐
  │      │       ┌───┴────┐      │      │NAT
  │      ├───────┤ switch ├──────┤      ├────
  │      │       └───┬────┘      │      │
  └──────┘           │           └──────┘
node1.tp4.b1         │         router.tp4.b1
                     │
                     │
                  ┌──┴───┐
                  │      │
                  │      │
                  │      │
                  └──────┘
                 dhcp.tp4.b1
```

> *Pour rappel, dans nos TPs avec VirtualBox, les switches sont les réseaux host-only (réseau privé hôte). **Assurez-vous de bien avoir désactiver le DHCP fourni par VirtualBox***.

### 2. Tableau d'adressage

| Machine         | LAN1            |
| --------------- | --------------- |
| `router.tp4.b1` | `10.4.1.254/24` |
| `dhcp.tp4.b1`   | `10.4.1.253/24` |
| `node1.tp4.b1`  | N/A             |
| `node2.tp4.b1`  | `10.4.1.12/24`  |

> *Pas d'adresse IP pour `node1.tp4.b1`, on le laisse de côté pour le moment.*

### 3. Setup topologie

➜ **Mettez en place la topologie**

- 3 VMs, un réseau host-only, des IPs statiques, on commence à répéter la même musique !
- il faut ajouter une carte NAT à `router.tp4.b1` pour lui donner un accès internet
- ajout de route par défaut sur `dhcp.tp4.b1` et`node2.tp4.b1` : `router.tp4.b1` doit être leur passerelle pour accéder à Internet

🌞 **Preuve de mise en place**
```
[et0@dhcp ~]$ ping apple.com
PING apple.com (17.253.144.10) 56(84) bytes of data.
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=1 ttl=56 time=20.3 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=2 ttl=56 time=21.5 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=3 ttl=56 time=17.8 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=4 ttl=56 time=19.8 ms
^C
--- apple.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3010ms
rtt min/avg/max/mdev = 17.848/19.877/21.517/1.321 ms
```
```
[et0@node2 ~]$ ping apple.com
PING apple.com (17.253.144.10) 56(84) bytes of data.
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=1 ttl=56 time=17.3 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=2 ttl=56 time=15.7 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=3 ttl=56 time=18.3 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=4 ttl=56 time=20.1 ms
^C
--- apple.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3011ms
rtt min/avg/max/mdev = 15.731/17.833/20.087/1.581 ms
```
```[et0@node2 ~]$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (10.4.1.254)  1.976 ms  1.670 ms  1.634 ms
 2  10.4.2.1 (10.4.2.1)  2.241 ms *  2.015 ms
 3  10.33.79.254 (10.33.79.254)  11.879 ms  11.863 ms  11.847 ms
 4  145.117.7.195.rev.sfr.net (195.7.117.145)  13.891 ms  13.856 ms  13.800 ms
 5  * * *
^C
```

### 4. Serveur DHCP

On va installer et configurer un serveur DHCP sur la machine `dhcp.tp4.b1`.

🌞 **Rendu**
```
[et0@dhcp ~]$ sudo dnf -y install dhcp-server
Last metadata expiration check: 0:09:53 ago on Fri Nov 17 11:41:32 2023.
Package dhcp-server-12:4.4.2-18.b1.el9.aarch64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```
```
[et0@dhcp ~]$ sudo vi /etc/dhcp/dhcpd.conf
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
# create new
# specify domain name
option domain-name     "srv.world";
# specify DNS server's hostname or IP address
option domain-name-servers     dlp.srv.world;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.4.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.4.1.137 10.4.1.237;
    # specify broadcast address
    option broadcast-address 10.4.1.255;
    # specify gateway
    option routers 10.4.1.254;
}
```
```
[et0@dhcp ~]$ sudo systemctl enable --now dhcpd
```
```
[et0@dhcp ~]$ sudo firewall-cmd --add-service=dhcp
[et0@dhcp ~]$ sudo firewall-cmd --runtime-to-permanent
```
```
[et0@dhcp ~]$ ll /var/lib/dhcpd
total 8
-rw-r--r--. 1 dhcpd dhcpd 285 Nov 17 11:46 dhcpd.leases
-rw-r--r--. 1 dhcpd dhcpd 221 Nov 17 11:45 dhcpd.leases~
-rw-r--r--. 1 dhcpd dhcpd   0 Apr 20  2023 dhcpd6.leases
```
```
[et0@dhcp ~]$ cat /var/lib/dhcpd/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.2b1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

server-duid "\000\001\000\001,\352\001\020b\334\312\275\013M";
```
```
[et0@dhcp ~]$ sudo systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
     Active: active (running) since Fri 2023-11-17 11:46:40 CET; 9min ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 1317 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 4263)
     Memory: 4.6M
        CPU: 5ms
     CGroup: /system.slice/dhcpd.service
             └─1317 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Nov 17 11:46:40 dhcp.tp4.b1 dhcpd[1317]: Config file: /etc/dhcp/dhcpd.conf
Nov 17 11:46:40 dhcp.tp4.b1 dhcpd[1317]: Database file: /var/lib/dhcpd/dhcpd.leases
Nov 17 11:46:40 dhcp.tp4.b1 dhcpd[1317]: PID file: /var/run/dhcpd.pid
Nov 17 11:46:40 dhcp.tp4.b1 dhcpd[1317]: Source compiled to use binary-leases
Nov 17 11:46:40 dhcp.tp4.b1 dhcpd[1317]: Wrote 0 leases to leases file.
Nov 17 11:46:40 dhcp.tp4.b1 dhcpd[1317]: Listening on LPF/enp0s1/62:dc:ca:bd:0b:4d/10.4.1.0/24
Nov 17 11:46:40 dhcp.tp4.b1 dhcpd[1317]: Sending on   LPF/enp0s1/62:dc:ca:bd:0b:4d/10.4.1.0/24
Nov 17 11:46:40 dhcp.tp4.b1 dhcpd[1317]: Sending on   Socket/fallback/fallback-net
Nov 17 11:46:40 dhcp.tp4.b1 dhcpd[1317]: Server starting service.
Nov 17 11:46:40 dhcp.tp4.b1 systemd[1]: Started DHCPv4 Server Daemon.
lines 1-23
```

> *Bon bah c'est pas tout mais c'est qu'il s'agirait de voir s'il fonctionne ce serveur DHCP !*

### 5. Client DHCP

➜ **Petite astuce**

- pour avoir un peu plus de détails sur l'interaction entre le client et votre serveur DHCP, vous pouvez lancer la commande `sudo journalctl -xe -u dhcpd -f` sur `dhcp.tp4.b1`
- cette commande permet de suivre en temps réel l'arrivée de nouveaux logs
- si vous laissez tourner cette commande pendant l'étape qui suit, vous allez voir arriver en temps réel les requêtes DHCP du client dnas les logs

🌞 **Test !**

```
[et0@node1 ~]$ sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s1

DEVICE=enp0s1

BOOTPROTO=dhcp
ONBOOT=yes
```
```
[et0@node1 ~]$ sudo nmcli con reload
```
```
[et0@node1 ~]$ sudo nmcli con up "System enp0s1"
```
```
[et0@node1 ~]$ sudo systemctl restart NetworkManager
```

🌞**Prouvez que**

```[et0@node1 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 9e:80:0d:d5:21:48 brd ff:ff:ff:ff:ff:ff
    inet 10.4.1.137/24 brd 10.4.1.255 scope global dynamic enp0s1
       valid_lft 559sec preferred_lft 559sec
    inet6 fe80::9c80:dff:fed5:2148/64 scope link 
       valid_lft forever preferred_lft forever
```
```
[et0@node1 ~]$ sudo nmcli con sh "enp0s1" | grep -i dhcp4
DHCP4.OPTION[3]:                        dhcp_lease_time = 600
DHCP4.OPTION[4]:                        dhcp_server_identifier = 10.4.1.253
DHCP4.OPTION[5]:                        domain_name = srv.world
DHCP4.OPTION[6]:                        expiry = 1700732378
```
```
Thursday 23 November 2023 09:29:38
```
```
Thursday 23 November 2023 09:39:38
```
```
[et0@node1 ~]$ ping 10.4.1.254
PING 10.4.1.254 (10.4.1.254) 56(84) bytes of data.
64 bytes from 10.4.1.254: icmp_seq=1 ttl=64 time=6.74 ms
64 bytes from 10.4.1.254: icmp_seq=2 ttl=64 time=4.39 ms
64 bytes from 10.4.1.254: icmp_seq=3 ttl=64 time=2.55 ms
64 bytes from 10.4.1.254: icmp_seq=4 ttl=64 time=2.43 ms
^C
--- 10.4.1.254 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3014ms
rtt min/avg/max/mdev = 2.430/4.029/6.740/1.747 ms
```
```
[et0@node1 ~]$ ping 10.4.1.12
PING 10.4.1.12 (10.4.1.12) 56(84) bytes of data.
64 bytes from 10.4.1.12: icmp_seq=1 ttl=64 time=6.16 ms
64 bytes from 10.4.1.12: icmp_seq=2 ttl=64 time=2.19 ms
64 bytes from 10.4.1.12: icmp_seq=3 ttl=64 time=2.25 ms
64 bytes from 10.4.1.12: icmp_seq=4 ttl=64 time=2.57 ms
^C
--- 10.4.1.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3013ms
rtt min/avg/max/mdev = 2.192/3.291/6.157/1.660 ms
```

🌞 **Bail DHCP serveur**

```
[et0@dhcp ~]$ cat /var/lib/dhcpd/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.2b1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

server-duid "\000\001\000\001,\361\332gb\334\312\275\013M";

lease 10.4.1.137 {
  starts 4 2023/11/23 09:29:38;
  ends 4 2023/11/23 09:39:38;
  cltt 4 2023/11/23 09:29:38;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 9e:80:0d:d5:21:48;
  uid "\001\236\200\015\325!H";
  client-hostname "node1";
}
```

### 6. Options DHCP

🌞 **Nouvelle conf !**

```
[et0@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
[sudo] password for et0: 
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
# create new
# specify domain name
option domain-name     8.8.8.8;
# specify DNS server's hostname or IP address
option domain-name-servers     dlp.srv.world;
# default lease time
default-lease-time 21600;
# max lease time
max-lease-time 25200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.4.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.4.1.137 10.4.1.237;
    # specify broadcast address
    option broadcast-address 10.4.1.255;
    # specify gateway
    option routers 10.4.1.254;
}
```
```
[et0@dhcp ~]$ sudo systemctl restart dhcpd
``````

🌞 **Test !**

```
[et0@node1 network-scripts]S sudo rm /var/lib/dhclient/dhclient.leases
```
```
[et0@node1 network-scripts]S sudo dhclient -v -r enp0s1
```
```
[et0@node1 ~]$ sudo cat /etc/resolv.conf
; generated by /usr/sbin/dhclient-script
search srv.world tp4.b1
nameserver 8.8.8.8
```
```
[et0@node1 ~]$ ip r s
default via 10.4.1.254 dev enp0s1 
10.4.1.0/24 dev enp0s1 proto kernel scope link src 10.4.1.138 
```
```
[et0@node1 ~]$ sudo cat /var/lib/dhclient/dhclient.leases
lease {
  interface "enp0s1";
  fixed-address 10.4.1.138;
  option subnet-mask 255.255.255.0;
  option routers 10.4.1.254;
  option dhcp-lease-time ⭐️ 21600 ⭐️;
  option dhcp-message-type 5;
  option domain-name-servers 8.8.8.8;
  option dhcp-server-identifier 10.4.1.253;
  option broadcast-address 10.4.1.255;
  option domain-name "srv.world";
  renew 4 2023/11/23 12:51:35;
  rebind 4 2023/11/23 15:30:47;
  expire 4 2023/11/23 16:15:47;
}
```
```
[et0@node1 ~]$ ping apple.com
PING apple.com (17.253.144.10) 56(84) bytes of data.
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=1 ttl=56 time=16.8 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=2 ttl=56 time=17.7 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=3 ttl=56 time=27.6 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=4 ttl=56 time=19.3 ms
^C
--- apple.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3013ms
rtt min/avg/max/mdev = 16.771/20.341/27.602/4.287 ms
```