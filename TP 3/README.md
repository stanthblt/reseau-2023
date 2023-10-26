# II. Routage

- [II. Routage](#ii-routage)
  - [1. Mise en place du routage](#1-mise-en-place-du-routage)
  - [2. Analyse de trames](#2-analyse-de-trames)
  - [3. Accès internet](#3-accès-internet)

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

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

- voir [le mémo Rocky](../../cours/memo/rocky_network.md) pour ça
- il faut taper une commande `ip route add` 
- il faut ajouter une seule route des deux côtés
- une fois les routes en place, vérifiez avec un `ping` que les deux machines peuvent se joindre (`john` et `marcel`)

Par exemple, la route de `john` pour joindre le réseau LAN2 de `marcel`, elle va ressembler à :

```bash
# 10.2.2.0/24 est l'adresse du réseau que john ne connaît pas
# 10.2.1.254 est l'adresse de la passerelle qu'il peut déjà joindre : votre router
10.2.2.0/24 via 10.2.1.254 dev enp0s3
```

![THE SIZE](./img/thesize.png)

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