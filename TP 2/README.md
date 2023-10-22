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

**Il vous faudra deux machines**, vous êtes libres :

- toujours possible de se connecter à deux avec un câble
- vous pouvez utiliser vos adresses IP WiFi sinon (RIP réseau d'YNOV)
- sinon, votre PC + une VM ça fait le taf aussi, c'est pareil
  - je peux aider sur le setup si besoin, comme d'hab
  - mais quand même, faire le TP à deux c'est toujours plus cool !

---

**Toutes les manipulations devront être effectuées depuis la ligne de commande.** Donc normalement, no screens toujours .

**Pour Wireshark, c'est pareil,** NO SCREENS. La marche à suivre :

- vous capturez le trafic que vous avez à capturer
- vous stoppez la capture (bouton carré rouge en haut à gauche)
- vous sélectionnez les paquets/trames intéressants (CTRL + clic)
- File > Export Specified Packets...
- dans le menu qui s'ouvre, cochez en bas "Selected packets only"
- sauvegardez, ça produit un fichier `.pcapng` (qu'on appelle communément "un ptit PCAP frer") que vous livrerez dans le dépôt git

**Si vous voyez le p'tit pote 🦈 c'est qu'il y a un PCAP à produire et à mettre dans votre dépôt git de rendu.**

# I. Setup IP

Le lab, il vous faut deux machines :

- les deux machines doivent être connectées physiquement
- vous devez choisir vous-mêmes les IPs à attribuer sur les interfaces réseau, les contraintes :
  - IPs privées (évidemment n_n)
  - dans un réseau qui peut contenir au moins 1000 adresses IP (il faut donc choisir un masque adapté)
  - oui c'est random, on s'exerce c'est tout, p'tit jog en se levant c:
  - le masque choisi doit être le plus grand possible (le plus proche de 32 possible) afin que le réseau soit le plus petit possible

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

- vous renseignerez dans le compte rendu :
  - les deux IPs choisies, en précisant le masque
  - l'adresse de réseau
  - l'adresse de broadcast
- vous renseignerez aussi les commandes utilisées pour définir les adresses IP *via* la ligne de commande

> Rappel : tout doit être fait *via* la ligne de commandes. Faites-vous du bien, et utilisez Powershell plutôt que l'antique cmd sous Windows svp.

🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

- un `ping` suffit !

🌞 **Wireshark it**

- `ping` ça envoie des paquets de type ICMP (c'est pas de l'IP, c'est un de ses frères)
  - les paquets ICMP sont encapsulés dans des trames Ethernet, comme les paquets IP
  - il existe plusieurs types de paquets ICMP, qui servent à faire des trucs différents
- **déterminez, grâce à Wireshark, quel type de paquet ICMP est envoyé par `ping`**
  - pour le ping que vous envoyez
  - et le pong que vous recevez en retour

> Vous trouverez sur [la page Wikipedia de ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol) un tableau qui répertorie tous les types ICMP et leur utilité

🦈 **PCAP qui contient les paquets ICMP qui vous ont permis d'identifier les types ICMP** (juste quelques-uns)

# II. ARP my bro

ARP permet, pour rappel, de résoudre la situation suivante :

- pour communiquer avec quelqu'un dans un LAN, il **FAUT** connaître son adresse MAC
- on admet un PC1 et un PC2 dans le même LAN :
  - PC1 veut joindre PC2
  - PC1 et PC2 ont une IP correctement définie
  - PC1 a besoin de connaître la MAC de PC2 pour lui envoyer des messages
  - **dans cette situation, PC1 va utilise le protocole ARP pour connaître la MAC de PC2**
  - une fois que PC1 connaît la mac de PC2, il l'enregistre dans sa **table ARP**

🌞 **Check the ARP table**

- utilisez une commande pour afficher votre table ARP
- déterminez la MAC de votre binome depuis votre table ARP
- déterminez la MAC de la *gateway* de votre réseau
  - celle de votre réseau physique, WiFi, genre YNOV, car il n'y en a pas dans votre ptit LAN
  - c'est juste pour vous faire manipuler un peu encore :)

> Il peut être utile de ré-effectuer des `ping` avant d'afficher la table ARP. En effet : les infos stockées dans la table ARP ne sont stockées que temporairement. Ce laps de temps est de l'ordre de ~60 secondes sur la plupart de nos machines.

🌞 **Manipuler la table ARP**

- utilisez une commande pour vider votre table ARP
- prouvez que ça fonctionne en l'affichant et en constatant les changements
- ré-effectuez des pings, et constatez la ré-apparition des données dans la table ARP

> Les échanges ARP sont effectuées automatiquement par votre machine lorsqu'elle essaie de joindre une machine sur le même LAN qu'elle. Si la MAC du destinataire n'est pas déjà dans la table ARP, alors un échange ARP sera déclenché.

🌞 **Wireshark it**

- vous savez maintenant comment forcer un échange ARP : il sufit de vider la table ARP et tenter de contacter quelqu'un, l'échange ARP se fait automatiquement
- mettez en évidence les deux trames ARP échangées lorsque vous essayez de contacter quelqu'un pour la "première" fois
  - déterminez, pour les deux trames, les adresses source et destination
  - déterminez à quoi correspond chacune de ces adresses

🦈 **PCAP qui contient les DEUX trames ARP**

> L'échange ARP est constitué de deux trames : un ARP broadcast et un ARP reply.

# III. DHCP

![YOU GET AN IP](./img/dhcp.jpg)

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