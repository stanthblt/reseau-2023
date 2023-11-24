# TP5 : TCP, UDP et services réseau

Dans ce TP on va explorer un peu les protocoles TCP et UDP. 

**La première partie est détente**, vous explorez TCP et UDP un peu, en vous servant de votre PC.

La seconde partie se déroule en environnement virtuel, avec des VMs. Les VMs vont nous permettre en place des services réseau, qui reposent sur TCP et UDP.  
**Le but est donc de commencer à mettre les mains de plus en plus du côté administration, et pas simple client.**

Dans cette seconde partie, vous étudierez donc :

- le protocole SSH (contrôle de machine à distance)
- le protocole DNS (résolution de noms)
  - essentiel au fonctionnement des réseaux modernes


# Sommaire

- [TP5 : TCP, UDP et services réseau](#tp5--tcp-udp-et-services-réseau)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
- [I. First steps](#i-first-steps)
- [II. Setup Virtuel](#ii-setup-virtuel)
  - [1. SSH](#1-ssh)
  - [2. Routage](#2-routage)
  - [3. Serveur Web](#3-serveur-web)

# 0. Prérequis

➜ **L'emoji 🖥️ indique une VM à créer**. Pour chaque VM, vous déroulerez la checklist suivante :

- [x] Créer la machine (avec une carte host-only)
- [x] Définir une IP statique à la VM
- [x] Donner un hostname à la machine
- [x] Utiliser SSH pour administrer la machine
- [x] Dès que le routeur est en place, n'oubliez pas d'ajouter une route par défaut aux autres VM pour qu'elles aient internet

# I. First steps

Faites-vous un petit top 5 des applications que vous utilisez sur votre PC souvent, des applications qui utilisent le réseau : un site que vous visitez souvent, un jeu en ligne, Spotify, j'sais po moi, n'importe.

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**


- `Apple : TCP` 
```
IP serveur : 17.253.144.10
Port serveur : 443
Port local : 56610
```
**Voir capture** : tp5_service_1.pcapng
- `YouTube : TCP`
```
IP serveur : 142.250.75.227
Port serveur : 443
Port local : 56682
```
**Voir capture** : tp5_service_2.pcapng
- `Netflix : TCP`
```
IP serveur : 54.74.73.31
Port serveur : 443
Port local : 56711
```
**Voir capture** : tp5_service_3.pcapng
- `Gmail : TCP`
```
IP serveur : 216.58.214.69
Port serveur : 443
Port local : 56816
```
**Voir capture** : tp5_service_4.pcapng
- `Discord : TCP`
```
IP serveur : 172.217.20.195
Port serveur : 443
Port local : 56837
```
**Voir capture** : tp5_service_5.pcapng

🌞 **Demandez l'avis à votre OS**

`stanislasthabault@MacBook-Pro-de-Stanislas ~ % sudo netstat -p tcp`

- `Apple` 
```
tcp4       0      0  172.20.10.4.56610      17.253.144.10.https    ESTABLISHED
```
- `YouTube` 
```
tcp4       0      0  172.20.10.4.56682      142.250.75.227.https    ESTABLISHED
```
- `Netflix` 
```
tcp4       0      0  172.20.10.4.56711      54.74.73.31.https    ESTABLISHED
```
- `Gmail` 
```
tcp4       0      0  172.20.10.4.56816      216.58.214.69.https    ESTABLISHED
```
- `Discord` 
```
tcp4       0      0  172.20.10.4.56837      172.217.20.195.https    ESTABLISHED
```

# II. Setup Virtuel

## 1. SSH

| Machine        | Réseau `10.5.1.0/24` |
| -------------- | -------------------- |
| `node1.tp5.b1` | `10.5.1.11`          |

🖥️ **Machine `node1.tp5.b1`**
$

Connectez-vous en SSH à votre VM.

🌞 **Examinez le trafic dans Wireshark**

- **déterminez si SSH utilise TCP ou UDP**
```
Transmission Control Protocol, Src Port: 22, Dst Port: 57284, Seg: 0, Ack: 1, Len: 0
```
- **repérez le *3-Way Handshake* à l'établissement de la connexion**
    -  les 3 premières lignes 
        -  `[SYN, ECE, CWR]`
        -  `[SYN, ACK, ECE]`
        -  `[ACK]`
- **repérez du trafic SSH**
    - les lignes 4 et 6 qui contiennent "SSHv2"
- **repérez le FIN ACK à la fin d'une connexion**
    - les lignes 8 et 9 qui contiennent
        - `[FIN, ACK]`

🌞 **Demandez aux OS**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % netstat | grep ssh
tcp4       0      0  10.5.1.1.57731         10.5.1.11.ssh          ESTABLISHED

```
```
[et0@node1 ~]$ ss | grep ssh
tcp   ESTAB 0      52                        10.5.1.11:ssh       10.5.1.1:57703       
```

🦈 **`tp5_3_way.pcapng` : une capture clean avec le 3-way handshake, un peu de trafic au milieu et une fin de connexion**

**Voir capture** : tp5_3_way

## 2. Routage

| Machine         | Réseau `10.5.1.0/24` |
| --------------- | -------------------- |
| `node1.tp5.b1`  | `10.5.1.11`          |
| `router.tp5.b1` | `10.5.1.254`         |

🖥️ **Machine `router.tp5.b1`**

🌞 **Prouvez que**

```
[et0@node1 ~]$ ip r s
default via 10.5.1.254 dev enp0s1 
10.5.1.0/24 dev enp0s1 proto kernel scope link src 10.5.1.11 metric 100 
```
```
[et0@node1 ~]$ ping apple.com
PING apple.com (17.253.144.10) 56(84) bytes of data.
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=1 ttl=52 time=54.5 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=2 ttl=52 time=33.1 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=3 ttl=52 time=81.6 ms
64 bytes from www.brkgls.com (17.253.144.10): icmp_seq=4 ttl=52 time=75.5 ms
^C64 bytes from 17.253.144.10: icmp_seq=5 ttl=52 time=55.7 ms

--- apple.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 33338ms
rtt min/avg/max/mdev = 33.091/60.073/81.634/17.219 ms
```


## 3. Serveur Web

Dans cette section on va monter un p'tit serveur Web sur une VM. Le serveur web hébergera un unique site web, qui sera super nul super moche parce qu'on est pas en cours de dév web :D

Le nom du serveur web qu'on va utiliser c'est NGINX. On l'installe, on le configure, on le lance, et PAF un site web hébergé en local disponible.

| Machine         | Réseau `10.5.1.0/24` |
| --------------- | -------------------- |
| `node1.tp5.b1`  | `10.5.1.11`          |
| `router.tp5.b1` | `10.5.1.254`         |
| `web.tp5.b1`    | `10.5.1.12`          |

🖥️ **Machine `web.tp5.b1`**

🌞 **Installez le paquet `nginx`**

```
[et0@web ~]$ sudo dnf install nginx
Last metadata expiration check: 0:09:29 ago on Fri Nov 24 09:56:34 2023.
Package nginx-1:1.20.1-14.el9_2.1.aarch64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

🌞 **Créer le site web**

```
[et0@web site_web_nul]$ pwd
/var/www/site_web_nul
```
```
[et0@web site_web_nul]$ sudo cat index.html
<!doctype html>
<html lang="fr">
<head>
  <meta charset="utf-8">
  <title>Titre de la page</title>
  <link rel="stylesheet" href="style.css">
  <script src="script.js"></script>
</head>
<body>
  ...
  <!-- Le reste du contenu -->
  <h1>Salut c'est moi</h1>
  ...
</body>
</html>
``````

🌞 **Donner les bonnes permissions**

```
[et0@web www]$ sudo chown -R nginx:nginx /var/www/site_web_nul
```

🌞 **Créer un fichier de configuration NGINX pour notre site web**

```
[et0@web conf.d]$ sudo cat site_web_nul.conf
server {
  # le port sur lequel on veut écouter
  listen 80;

  # le nom de la page d'accueil si le client de la précise pas
  index index.html;

  # un nom pour notre serveur (pas vraiment utile ici, mais bonne pratique)
  server_name www.site_web_nul.b1;

  # le dossier qui contient notre site web
  root /var/www/site_web_nul;
}
```

🌞 **Démarrer le serveur web !**

```
[et0@web ~]$ sudo systemctl start nginx
```
```
[et0@web ~]$ sudo systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: d>
     Active: active (running) since Fri 2023-11-24 10:22:06 CET; 2min 56s ago
    Process: 11464 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, sta>
    Process: 11465 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCE>
    Process: 11466 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 11467 (nginx)
      Tasks: 3 (limit: 4263)
     Memory: 2.8M
        CPU: 27ms
     CGroup: /system.slice/nginx.service
             ├─11467 "nginx: master process /usr/sbin/nginx"
             ├─11468 "nginx: worker process"
             └─11469 "nginx: worker process"

Nov 24 10:22:06 web.tp5.b1 systemd[1]: Starting The nginx HTTP and reverse prox>
Nov 24 10:22:06 web.tp5.b1 nginx[11465]: nginx: the configuration file /etc/ngi>
Nov 24 10:22:06 web.tp5.b1 nginx[11465]: nginx: configuration file /etc/nginx/n>
Nov 24 10:22:06 web.tp5.b1 systemd[1]: Started The nginx HTTP and reverse proxy>
lines 1-19/19 (END)
```

🌞 **Ouvrir le port firewall**

```
[et0@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
```
```
[et0@web ~]$ sudo firewall-cmd --reload
success
```
```
[et0@web ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s1
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 80/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

🌞 **Visitez le serveur web !**
```
[et0@node1 ~]$ curl http://10.5.1.12
<!doctype html>
<html lang="fr">
<head>
  <meta charset="utf-8">
  <title>Titre de la page</title>
  <link rel="stylesheet" href="style.css">
  <script src="script.js"></script>
</head>
<body>
  ...
  <!-- Le reste du contenu -->
  <h1>Salut c'est moi</h1>
  ...
</body>
</html>
```

🌞 **Visualiser le port en écoute**

```
[et0@web ~]$ sudo ss -ltunp
tcp            LISTEN          0               511                            0.0.0.0:80                          0.0.0.0:*             users:(("nginx",pid=11469,fd=6),("nginx",pid=11468,fd=6),("nginx",pid=11467,fd=6))
```

🌞 **Analyse trafic**

**Voir capture** : tp5_web.pcapng

  - `le 3 way handshake TCP`
    - les 3 premières lignes SYN, SYN ACK, ACK
  - `du trafic HTTP`
    - ligne 4 : GET / HTTP/1.1
  - `le contenu de la page HTML retourné (oui c'est visible direct dans Wireshark)`
    - ligne 6 : HTTP/1.1

