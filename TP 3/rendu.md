# TP3 : On va router des trucs

## Sommaire

- [TP3 : On va router des trucs](#tp3--on-va-router-des-trucs)
  - [Sommaire](#sommaire)
  - [0. Prérequis](#0-prérequis)
  - [I. ARP](#i-arp)
  - [II. Routage](#ii-routage)
  - [III. DHCP](#iii-dhcp)

## I. [ARP](#i-arp)

## II. [Routage](#ii-routage)

## III. [DHCP](#iii-dhcp)

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

- `john` peut ping `router` sur `10.3.1.254`
- `marcel` peut ping `router` sur `10.3.2.254`
- une fois que ces deux `ping` passent, go à la suite !

> C'est normal si `john` et `marcel` ne peuvent pas se ping encore ! En effet, ni l'un ni l'autre ne connaît de route pour aller dans le réseau de l'autre. `john` n'a aucune idée de comment joindre une adresse IP qui se trouve dans le LAN2.

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

```

## 2. Analyse de trames

🌞**Analyse des échanges ARP**

- videz les tables ARP des trois noeuds
- effectuez un `ping` de `john` vers `marcel`
- essayez de déduire les échanges ARP qui ont eu lieu
  - en regardant les tables ARP des 3 machines
  - en lançant `tcpdump` pour capturer le trafic et l'analyser
- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames** utiles pour l'échange (ARP et ping/pong)

Par exemple (copiez-collez ce tableau ce sera le plus simple) :

| ordre | type trame  | IP source | MAC source                | IP destination | MAC destination            |
| ----- | ----------- | --------- | ------------------------- | -------------- | -------------------------- |
| 1     | Requête ARP | x         | `marcel` `AA:BB:CC:DD:EE` | x              | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | x         | ?                         | x              | `marcel` `AA:BB:CC:DD:EE`  |
| ...   | ...         | ...       | ...                       |                |                            |
| ?     | Ping        | ?         | ?                         | ?              | ?                          |
| ?     | Pong        | ?         | ?                         | ?              | ?                          |

## 3. Accès internet

🌞**Donnez un accès internet à vos machines** - config routeur

- ajoutez une carte NAT (dans l'interface de Virtualbox) en 3ème inteface sur le `router` pour qu'il ait un accès internet
- une fois que c'est fait, vérifiez qu'il a bien un accès internet
- sous Rocky...
  - pour autoriser le routeur à router des paquets vers internet, et faire ça propre, ça nécessite quelques manips
  - pour qu'il agisse comme un routeur normal quoi
  - référez-vous au [mémo Rocky](../../cours/memo/rocky_network.md)

🌞**Donnez un accès internet à vos machines** - config clients

- ajoutez une route par défaut à `john` et `marcel` (voir [le mémo Rocky](../../cours/memo/rocky_network.md))
  - vérifiez que vous avez accès internet avec un `ping`
  - le `ping` doit être vers une IP, PAS un nom de domaine
- donnez leur aussi l'adresse d'un serveur DNS qu'ils peuvent utiliser (voir [mémo Rocky](../../cours/memo/rocky_network.md))
  - `ping` vers un nom de domaine pour vérifier la résolution de nom

🌞**Analyse de trames**

- effectuez un `ping` depuis `john` vers `marcel`
- capturez le ping depuis `router` avec `tcpdump`
  - faites deux captures : une sur chaque interface du routeur (celle qui est dans le LAN1 et celle qui est dans le LAN2)
- analysez un ping aller et le retour qui correspond et mettez dans un tableau :

| ordre | type trame | IP source            | MAC source                | IP destination | MAC destination |     |
| ----- | ---------- | -------------------- | ------------------------- | -------------- | --------------- | --- |
| 1     | ping       | `marcel` `10.3.1.12` | `marcel` `AA:BB:CC:DD:EE` | `8.8.8.8`      | ?               |     |
| 2     | pong       | ...                  | ...                       | ...            | ...             | ... |

> Avec les deux captures vous devez observer 4 trames pour chaque ping échangé : 1) trame ping entre `john` et `router` (capture1) 2) trame ping entre `router` et `marcel` (capture2) 3) trame pong entre `marcel` et `router` (capture1) 4) trame pong entre `router` et `john` (capture2)

🦈 **Capture réseau `tp3_routage_lan1.pcapng`**  
🦈 **Capture réseau `tp3_routage_lan2.pcapng`**