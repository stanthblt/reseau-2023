# TP1 - Premier pas réseau


# Sommaire
- [TP1 - Premier pas réseau](#tp1---premier-pas-réseau)
- [Sommaire](#sommaire)
- [I. Exploration locale en solo](#i-exploration-locale-en-solo)
  - [1. Affichage d'informations sur la pile TCP/IP locale](#1-affichage-dinformations-sur-la-pile-tcpip-locale)
    - [En ligne de commande](#en-ligne-de-commande)
    - [En graphique (GUI : Graphical User Interface)](#en-graphique-gui--graphical-user-interface)
  - [2. Modifications des informations](#2-modifications-des-informations)
    - [A. Modification d'adresse IP (part 1)](#a-modification-dadresse-ip-part-1)
- [II. Exploration locale en duo](#ii-exploration-locale-en-duo)
  - [Création du réseau (oupa)](#création-du-réseau-oupa)
  - [3. Modification d'adresse IP](#3-modification-dadresse-ip)
  - [4. Petit chat privé](#4-petit-chat-privé)
  - [5. Firewall](#5-firewall)
  - [6. Utilisation d'un des deux comme gateway](#6-utilisation-dun-des-deux-comme-gateway)
- [III. Manipulations d'autres outils/protocoles côté client](#iii-manipulations-dautres-outilsprotocoles-côté-client)
  - [1. DHCP](#1-dhcp)
  - [2. DNS](#2-dns)
- [IV. Wireshark](#iv-wireshark)
- [Bilan](#bilan)

# I. Exploration locale en solo

## 1. Affichage d'informations sur la pile TCP/IP locale

### En ligne de commande

**🌞 Affichez les infos des cartes réseau de votre PC**
``` 
    stanislasthabault@MacBook-Pro-de-Stanislas ~ % ifconfig            
    en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        options=6460<TSO4,TSO6,CHANNEL_IO,PARTIAL_CSUM,ZEROINVERT_CSUM>
        ether ⭐ bc:d0:74:58:6f:5e ⭐
        inet6 fe80::400:2d29:e34a:6618%en0 prefixlen 64 secured scopeid 0xe 
        inet ⭐ 10.33.48.11 ⭐ netmask 0xfffffc00 broadcast 10.33.51.255
        nd6 options=201<PERFORMNUD,DAD>
        media: autoselect
        status: active
    en7: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	options=6464<VLAN_MTU,TSO4,TSO6,CHANNEL_IO,PARTIAL_CSUM,ZEROINVERT_CSUM>
	ether ⭐ 22:e0:4c:a1:31:36 ⭐
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect (none)
	status: inactive
```
**🌞 Affichez votre gateway**
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % route -n get default
    route to: default
    destination: default
        mask: default
        gateway: ⭐ 10.33.51.254 ⭐
    interface: en0
        flags: <UP,GATEWAY,DONE,STATIC,PRCLONING,GLOBAL>
    recvpipe  sendpipe  ssthresh  rtt,msec    rttvar  hopcount      mtu     expire
        0         0         0         0         0         0      1500         0 
```
**🌞 Déterminer la MAC de la passerelle**
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % arp -n 10.33.51.254
    ? (10.33.51.254) at ⭐ 7c:5a:1c:cb:fd:a4 ⭐ on en0 ifscope [ethernet]
```

### En graphique (GUI : Graphical User Interface)

🌞 **Il est possible que vous perdiez l'accès internet.**
```
Si le réseau utilise un serveur DHCP pour attribuer automatiquement des adresses IP, le changement manuel peut entrer en conflit, et donc on peut perdre la connexion.
```
# II. Exploration locale en duo

## Création du réseau (oupa)

🌞 **Vérifier à l'aide d'une commande que votre IP a bien été changée**
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % ifconfig
    en7: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        options=6464<VLAN_MTU,TSO4,TSO6,CHANNEL_IO,PARTIAL_CSUM,ZEROINVERT_CSUM>
        ether 22:e0:4c:a1:31:36
        inet6 fe80::879:2f8c:7ab8:3e17%en7 prefixlen 64 secured scopeid 0x17 
        inet ⭐ 10.10.10.1 ⭐ netmask 0xffffff00 broadcast 10.10.10.255
        nd6 options=201<PERFORMNUD,DAD>
        media: autoselect (100baseTX <full-duplex>)
        status: active
```
🌞 **Vérifier que les deux machines se joignent**
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % ping 10.10.10.2
PING 10.10.10.2 (10.10.10.2): 56 data bytes
64 bytes from 10.10.10.2: icmp_seq=0 ttl=128 time=2.212 ms
64 bytes from 10.10.10.2: icmp_seq=1 ttl=128 time=2.834 ms
64 bytes from 10.10.10.2: icmp_seq=2 ttl=128 time=3.126 ms
64 bytes from 10.10.10.2: icmp_seq=3 ttl=128 time=3.130 ms
64 bytes from 10.10.10.2: icmp_seq=4 ttl=128 time=2.167 ms
64 bytes from 10.10.10.2: icmp_seq=5 ttl=128 time=3.133 ms
64 bytes from 10.10.10.2: icmp_seq=6 ttl=128 time=2.361 ms
64 bytes from 10.10.10.2: icmp_seq=7 ttl=128 time=2.427 ms
64 bytes from 10.10.10.2: icmp_seq=8 ttl=128 time=2.613 ms
64 bytes from 10.10.10.2: icmp_seq=9 ttl=128 time=3.265 ms
64 bytes from 10.10.10.2: icmp_seq=10 ttl=128 time=2.779 ms
^C
--- 10.10.10.2 ping statistics ---
10 packets transmitted, 10 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 1.981/2.639/3.265/0.347 ms
```
🌞 **Déterminer l'adresse MAC de votre correspondant**
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % arp -n 10.10.10.2
? (10.10.10.2) at ⭐ 2c:f0:5d:6c:11:b8 ⭐ on en7 ifscope [ethernet]
```
## 4. Petit chat privé

🌞 **Visualiser la connexion en cours**
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % netstat -a -n 
    tcp4       0      0  10.10.10.1.51752       10.10.10.2.8888        ESTABLISHED
```
🌞 **Pour aller un peu plus loin**
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % nc 10.10.10.2 8888   
```
## 5. Firewall

🌞 **Activez et configurez votre firewall**
- Ping
```
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /sbin/ping
```
- nc
```
sudo nano /etc/pf.conf
pass in proto tcp from any to any port 8888
sudo pfctl -f /etc/pf.conf
```
## 6. Utilisation d'un des deux comme gateway

🌞 **Tester l'accès internet**

# III. Manipulations d'autres outils/protocoles côté client

## 1. DHCP

🌞 **Exploration du DHCP, depuis votre PC**

🌞 **Trouver l'adresse IP du serveur DNS que connaît votre ordinateur**
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % cat /etc/resolv.conf    
#
# macOS Notice
#
# This file is not consulted for DNS hostname resolution, address
# resolution, or the DNS query routing mechanism used by most
# processes on this system.
#
# To view the DNS configuration used by this system, use:
#   scutil --dns
#
# SEE ALSO
#   dns-sd(1), scutil(8)
#
# This file is automatically generated.
#
nameserver ⭐ 172.20.10.1 ⭐
```
🌞 **Utiliser, en ligne de commande l'outil `nslookup` (Windows, MacOS) ou `dig` (GNU/Linux, MacOS) pour faire des requêtes DNS à la main**
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % nslookup google.com        
Server:		⭐ 172.20.10.1 ⭐
Address:	172.20.10.1#53

Non-authoritative answer:
Name:	google.com
Address: 216.58.213.78
```
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % nslookup ynov.com  
Server:		⭐ 172.20.10.1 ⭐
Address:	172.20.10.1#53

Non-authoritative answer:
Name:	ynov.com
Address: 104.26.11.233
Name:	ynov.com
Address: 172.67.74.226
Name:	ynov.com
Address: 104.26.10.233
```
```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % nslookup 231.34.113.12      
Server:		172.20.10.1
Address:	172.20.10.1#53

** server can't find 12.113.34.231.in-addr.arpa: NXDOMAIN
```
⭐ Le serveur DNS n'a pas pu trouver d'enregistrement associé à l'adresse IP ⭐
```
Server:		172.20.10.1
Address:	172.20.10.1#53

Non-authoritative answer:
17.2.34.78.in-addr.arpa	name = cable-78-34-2-17.nc.de.

Authoritative answers can be found from:
```
⭐ Le serveur DNS a pu trouver un enregistrement associé à l'adresse IP 17.2.34.78 ⭐

# IV. Wireshark

🌞 **Utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence :**

# Bilan

**Vu pendant le TP :**

- visualisation de vos interfaces réseau (en GUI et en CLI)
- extraction des informations IP
  - adresse IP et masque
  - calcul autour de IP : adresse de réseau, etc.
- connaissances autour de/aperçu de :
  - un outil de diagnostic simple : `ping`
  - un outil de scan réseau : `nmap`
  - un outil qui permet d'établir des connexions "simples" (on y reviendra) : `netcat`
  - un outil pour faire des requêtes DNS : `nslookup` ou `dig`
  - un outil d'analyse de trafic : `wireshark`
- manipulation simple de vos firewalls

**Conclusion :**

- Pour permettre à un ordinateur d'être connecté en réseau, il lui faut **une liaison physique** (par câble ou par *WiFi*).  
- Pour réceptionner ce lien physique, l'ordinateur a besoin d'**une carte réseau**. La carte réseau porte une adresse MAC  
- **Pour être membre d'un réseau particulier, une carte réseau peut porter une adresse IP.**
Si deux ordinateurs reliés physiquement possèdent une adresse IP dans le même réseau, alors ils peuvent communiquer.  
- **Un ordintateur qui possède plusieurs cartes réseau** peut réceptionner du trafic sur l'une d'entre elles, et le balancer sur l'autre, servant ainsi de "pivot". Cet ordinateur **est appelé routeur**.
- Il existe dans la plupart des réseaux, certains équipements ayant un rôle particulier :
  - un équipement appelé *passerelle*. C'est un routeur, et il nous permet de sortir du réseau actuel, pour en joindre un autre, comme Internet par exemple
  - un équipement qui agit comme **serveur DNS** : il nous permet de connaître les IP derrière des noms de domaine
  - un équipement qui agit comme **serveur DHCP** : il donne automatiquement des IP aux clients qui rejoigne le réseau
  - **chez vous, c'est votre Box qui fait les trois :)**
