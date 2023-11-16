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

- depuis `dhcp.tp4.b1`, envoyer un `ping` vers un nom de domaine public (pas une IP)
- depuis `node2.tp4.b1`, envoyer un `ping` vers un nom de domaine public (pas une IP)
- depuis `node2.tp4.b1`, un `traceroute` vers une IP publique pour montrer que vos paquets à destination d'internet passent bien par le `router.tp4.b1`

### 4. Serveur DHCP

On va installer et configurer un serveur DHCP sur la machine `dhcp.tp4.b1`.

➜ C'est un setup assez simple, je ne vais pas réinventer la roue et je préfère que vous commenciez à pratiquer les docs qu'on trouve en ligne.

[Ce lien](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=dhcp&f=1) est cool, c'est le strict minimum, sans fioriture.

Quelques notes pour vous aider à appréhender le truc :

- **la commande `dnf install`** permet d'ajouter un paquet sur le système
  - on installe un logiciel quoi !
  - là vous allez installer le serveur DHCP (un logiciel comme un autre)
- ensuite vous allez modifier **le fichier de configuration `/etc/dhcp/dhcpd.conf`**
  - essayez de comprendre ce fichier : c'est à votre portée
  - **enlevez les commentaires** en anglais de votre fichier, commentez-le à votre sauce si vous voulez
  - les commentaires sont les lignes qui commencent par `#`
- **la commande `systemctl enable --now dhcpd`** permet de :
  - démarrer le serveur DHCP
  - activer son démarrage automatique au boot de la machine
- en cas de soucis lors du démarrage
  - **lisez bien le message d'erreur**
  - essayez de l'interpréter et d'agir en conséquence
  - sinon, call me !

🌞 **Rendu**

- toutes les commandes tapées pour monter votre serveur DHCP sur `dhcp.tp4.b1`
- un `systemctl status dhcpd` qui affiche l'état du serveur (je dois voir qu'il est actif)
- je veux aussi un `cat /etc/dhcp/dhcpd.conf` dans le compte-rendu, pour que je vois le fichier de configuration
  - habituez-vous à me montrer vos fichiers de conf avec la commande `cat` dans les compte-rendus

> *Bon bah c'est pas tout mais c'est qu'il s'agirait de voir s'il fonctionne ce serveur DHCP !*

### 5. Client DHCP

➜ **Petite astuce**

- pour avoir un peu plus de détails sur l'interaction entre le client et votre serveur DHCP, vous pouvez lancer la commande `sudo journalctl -xe -u dhcpd -f` sur `dhcp.tp4.b1`
- cette commande permet de suivre en temps réel l'arrivée de nouveaux logs
- si vous laissez tourner cette commande pendant l'étape qui suit, vous allez voir arriver en temps réel les requêtes DHCP du client dnas les logs

🌞 **Test !**

- utilisez `node1.tp4.b1` pour faire les tests : il va récupérer une IP avec votre serveur
- référez-vous au [mémo](../../cours/memo/rocky_network.md) pour voir comment configurer une interface pour qu'elle récupère une IP dynamiquement en DHCP

🌞**Prouvez que**

- `node1.tp4.b1` a bien récupéré une IP **dynamiquement**
- `node1.tp4.b1` a enregistré un bail DHCP
  - déterminer la date exacte de création du bail
  - déterminer la date exacte d'expiration
  - déterminer l'adresse IP du serveur DHCP (depuis `node1.tp4.b1` : il a enregistré l'adresse IP du serveur DHCP)
- vous pouvez ping `router.tp4.b1` et `node2.tp4.b1` grâce à cette nouvelle IP récupérée

🌞 **Bail DHCP serveur**

- sur `dhcp.tp4.b1` montrez le fichier qui contient le bail DHCP de `node1.tp4.b1`

### 6. Options DHCP

Dans cette partie, vous allez modifier la conf de votre serveur DHCP. Vous allez utiliser les deux options suivantes dans la conf :

- `option routers x.x.x.x;`
  - permet de préciser l'IP de la passerelle du réseau au client
  - remplacer `x.x.x.x` par l'adresse IP de `router.tp4.b1`
- `option domain-name-servers x.x.x.x;`
  - permet de préciser au client l'adresse IP d'un serveur DNS joignable depuis ce réseau
  - remplacer `x.x.x.x` par l'adresse IP d'un serveur DNS public que vous connaissez
- `default-lease-time xxx;` et `max-lease-time xxx;`
  - qui permettent de modifier la durée du bail DHCP
  - `xxx` est une valeur en seconde
  - vous devrez indiquer une durée de bail de 6 heures

🌞 **Nouvelle conf !**

- montrez la nouvelle conf (avec la commande `cat`)
- redémarrage du service DHCP (`sudo systemctl restart dhcpd`)

🌞 **Test !**

- redemandez une IP avec le client `node1.tp4.b1`
- prouvez-que :
  - vous avez enregistré l'adresse d'un serveur DNS
    - sous Linux, on consulte le serveur DNS actuel en affichant le contenu du fichier `/etc/resolv.conf`
  - vous avez une nouvelle route par défaut qui a été récupérée dynamiquement
  - la durée de votre bail DHCP est bien de 6 heures
- prouvez que vous avez un accès Internet après cet échange DHCP

🌞 **Capture Wireshark**

- utilisez `tcpdump` pour capturer un échange DHCP complet entre `node1.tp4.b1` et `dhcp.tp4.b1`

🦈 **`tp4_dhcp_server.pcapng`**

➜ **Un vrai serveur DHCP** qui donne tout ce qu'il faut aux clients pour qu'ils aient un accès au LAN (une adresse IP) et un accès internet en plus (l'adresse de la passerelle et l'adresse d'un serveur DNS joignable).