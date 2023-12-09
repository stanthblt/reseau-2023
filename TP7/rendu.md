TP7 : Do u secure
Dans ce TP on va aborder plusieurs cas d'application de la cryptographie en informatique.
On va donc voir ici 3 cas d'application :

SSH

le serveur prouve son identité aux gens qui se connectent
le client utilise une paire de clé plutôt qu'un apssword pour se co


HTTPS

le serveur prouve son identité avec un certificat (qui contient sa clé publique)
le client et le serveur établissent une connexion chiffrée avec chacun une paire de clés, dans le but de délivrer du trafic HTTP


bonus : VPN

le serveur prouve son identité au client
le client prouve son identité au serveur
le client et le serveur établissent une connexion chiffrée avec chacun une paire de clés, dans le but que le client accès à un LAN à distance


Sommaire


TP7 : Do u secure

Sommaire


0. Setup
I. Setup LAN
II. SSH
III. Web
IV. Bonus : VPN


0. Setup
➜ Pour chaque VM, vous déroulerez la checklist suivante :


 Créer la machine (avec une carte host-only)

 Définir une IP statique à la VM

 Donner un hostname à la machine

 Utiliser SSH pour administrer la machine

 Remplir votre fichier hosts, celui de votre PC, pour accéder au VM avec un nom

 Dès que le routeur est en place, n'oubliez pas d'ajouter une route par défaut aux autres VM pour qu'elles aient internet


I. Setup LAN



Name
LAN1 10.7.1.0/24





router.tp7.b1
10.7.1.254


john.tp7.b1
10.7.1.11



🖥️ Machine router.tp7.b1

ajoutez lui aussi une carte NAT en plus de la carte host-only (privé hôte en français) pour qu'il ait un accès internet
toutes les autres VMs du TP devront utiliser ce routeur pour accéder à internet
n'oubliez pas d'activer le routage vers internet sur cette machine :


$ sudo firewall-cmd --add-masquerade --permanent
$ sudo firewall-cmd --reload


🖥️ Machine john.tp7.b1

une machine qui servira de client au sein du réseau pour effectuer des tests
suivez-bien la checklist !
testez tout de suite avec john que votre routeur fonctionne et que vous avez un accès internet


# II. SSH

Dans cette section on va s'intéresser à la place du chiffrement dans l'utilisation du protocole SSH.

En particulier, on va :

- en tant que client qui se connecte, vérifier qu'on se connecte au bon serveur SSH
  - cool pour éviter les attaques man-in-the-middle par exemple
- en tant que client, utiliser une clé plutôt qu'un password pour se connecter au serveur
  - on évite l'erreur humaine
  - on évite l'échange d'un password qui est une donnée sensible à travers le réseau
  - avec une clé, à aucun moment une donnée sensible ne circule sur le réseau
- en profiter pour faire un peu de conf du serveur SSH, lié au réseau

## Sommaire

- [II. SSH](#ii-ssh)
  - [Sommaire](#sommaire)
  - [0. Setup](#0-setup)
  - [1. Fingerprint](#1-fingerprint)
    - [A. Explications](#a-explications)
    - [B. Manips](#b-manips)
  - [2. Conf serveur SSH](#2-conf-serveur-ssh)
  - [3. Connexion par clé](#3-connexion-par-clé)
    - [A. Explications](#a-explications-1)
    - [B. Manips](#b-manips-1)
    - [C. Changement de fingerprint](#c-changement-de-fingerprint)

![SSH keys](./img/use_ssh_key.png)

## 0. Setup

| Name            | LAN1 `10.7.1.0/24` |
| --------------- | ------------------ |
| `router.tp7.b1` | `10.7.1.254`       |
| `john.tp7.b1`   | `10.7.1.11`        |

Pas de nouvelles machines pour cette section donc.

## 1. Fingerprint

### A. Explications

💡 **Cette section explique comment vous pouvez vous assurer que vous vous connectez au bon serveur SSH.**

Qu'un vilain hacker n'est pas entre vous et le serveur SSH (*man-in-the-middle*) par exemple.

➜ **Lorsqu'on se connecte à une machine en SSH pour la première fois, apparaît un message comme celui-ci :**

```bash
The authenticity of host '10.7.1.103 (10.7.1.103)' can't be established.
ED25519 key fingerprint is SHA256:CoMGKXc0JWXwmMEiVCxxJ7SifJW18MfCLGMJKJMNO9A.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Ce message nous informe de l'empreinte (ou *fingerprint* en anglais) du serveur : c'est une chaîne de caractère qui identifie de façon unique le serveur.  
Dans l'exemple au dessus, le serveur `10.7.1.103` me présente son empreinte : `CoMGKXc0JWXwmMEiVCxxJ7SifJW18MfCLGMJKJMNO9A`.

> *Certains serveurs SSH ont plusieurs clés pour différents contextes, et donc plusieurs empreintes. Ici, le serveur nous indique précisément qu'il nous présente le hash en `sha256` de sa clé qui utilise l'algorithme `ED25519`.*

Je peux donc choisir de faire confiance, de trust, cette empreinte et saisir `yes`. A l'inverse je peux saisir `no` si je trust pas et ainsi annuler la connexion.

➜ **Comment trust ?**

Il faut se connecter manuellement au serveur, sans SSH donc. Dans notre cas, utilisez la console de la VM directement.  
On peut alors utiliser une commande pour consulter l'empreinte du serveur directement :

> *Comme indiqué au dessus, ce qu'on va vérifier, c'est précisément l'empreinte `sha256` de sa clé en `ED25519`.*

```bash
# on regarde les clés du serveur qui existe
$ ls /etc/ssh

# on peut alors calculer l'empreinte sha256 d'une clé spécifique avec ssh-keygen -l
$ ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key
```

➜ **Le message n'apparaît plus aux connexions suivantes**

- c'est uniquement à la première connexion ce message
- après, ça s'affiche plus jamais, car votre PC a enregistré l'empreinte
- à chaque connexion suivante, il vérifie que l'empreinte est la bonne automatiquement
- le fichier qui stocke les empreintes que vous connaissez est appelé `known_hosts`

> C'est dans le nom `known_hosts` : les hôtes connus, littéralement !

`known_hosts` est un simple fichier texte qui se trouve dans votre répertoire personnel, dans un sous-dossier `.ssh` :

```bash
# Avec un OS Linux
$ ls /home/<USER>/.ssh

# Windows
$ ls C:/Users/<USER>/.ssh

# MacOS je suppose que c'est 
$ ls /Users/<USER>/.ssh
```

### B. Manips

> **Si tu t'es déjà connecté à `john` en SSH, tu n'auras pas le message de première connexion.** Il faudra donc éditer ton fichier `known_hosts`, supprimer la ligne qui concerne `john` afin d'avoir de nouveau le message du serveur qui présente son fingerprint, comme à a la première connexion.

🌞 **Effectuez une connexion SSH en vérifiant le fingerprint**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % ssh et0@10.7.1.11
The authenticity of host '10.7.1.11 (10.7.1.11)' can't be established.
ED25519 key fingerprint is SHA256:8G9oZtmwVTJkYPNfciTXghADmx83nHt2lk6Zej1RxVw.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.7.1.11' (ED25519) to the list of known hosts.
et0@10.7.1.11's password: 
Last login: Sun Dec  3 23:01:10 2023
```

## 2. Conf serveur SSH

On va faire un truc très basique, pour manipuler un peu toujours les cartes réseau, les IP, les ports, toussa : on va choisir explicitement l'IP et le port où tourne notre serveur SSH sur `router`.

La configuration du serveur SSH se fait dans le fichier `/etc/ssh/sshd_config`, utilisez `nano` pour le modifier par exemple.

Il faut être admin pour modifier le fichier, il faudra donc préfixer votre commande avec `sudo`.

🌞 **Consulter l'état actuel**

```
[et0@routeur ~]$ sudo ss -alntpu
Netid   State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port   Process                                                                           
tcp     LISTEN   0        128              0.0.0.0:22            0.0.0.0:*       users:(("sshd",pid=651,fd=3))                                                  
tcp     LISTEN   0        128                 [::]:22               [::]:*       users:(("sshd",pid=651,fd=4))
```

🌞 **Modifier la configuration du serveur SSH**

```
[et0@routeur ~]$ sudo vi /etc/ssh/sshd_config
# To modify the system-wide sshd configuration, create a  *.conf  file under
#  /etc/ssh/sshd_config.d/  which will be automatically included below
Include /etc/ssh/sshd_config.d/*.conf

# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change. 
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
Port 22000 
#AddressFamily any 
ListenAddress 10.7.1.254
#ListenAddress ::
```

🌞 **Prouvez que le changement a pris effet**

```
[et0@routeur ~]$ sudo ss -alntpu
Netid   State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port   Process                                                                           
tcp     LISTEN   0        128              10.7.1.254:22            0.0.0.0:*       users:(("sshd",pid=651,fd=3))                                                  
tcp     LISTEN   0        128                 [::]:22               [::]:*       users:(("sshd",pid=651,fd=4))
```

🌞 **N'oubliez pas d'ouvrir ce nouveau port dans le firewall**

```
[et0@routeur ~]$ sudo firewall-cmd --add-port=22000/tcp --permanent
Warning: ALREADY_ENABLED: 22000:tcp
success
```

🌞 **Effectuer une connexion SSH sur le nouveau port**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % ssh et0@10.7.1.254 -p 22000
et0@10.7.1.254's password: 
Last login: Sun Dec  3 23:24:33 2023 from 10.7.1.1
```

```
$ ssh toto@router.tp7.b1 -p <PORT>
```

## 3. Connexion par clé

### A. Explications

💡 **Cette section explique comment vous pouvez remplacer l'utilisation d'un password (faible en terme de sécurité) par une clé (fort en terme de sécurité).**

Quand on se connecte à une machine (genre toi quand t'ouvres ton windows le matin, ou quand tu te co à une machine en SSH, c'est la même chose), l'OS nous demande de saisir un password pour ouvrir une session avec un utilisateur de la machine. Rien de nouveau sous le soleil mao.

Il se trouve qu'en terme de sécurité, en particulier pour des connexions à distance comme avec SSH, **bah c'est claqué l'utilisation d'un password**.  
On fait circuler le password sur le réseau, alors qu'il est hyper sensible, c'est nul ça.  
Puis tu dois t'en souvenir, ou le stocker dans une application. C'est CLAQUE.

**Pour augmenter le niveau de sécurité de la connexion, on préfère remplacer l'utilisation d'un password par l'utilisation d'une paire de clé.**

Nous on est juste des utilisateurs de ça, personne vous demande de comprendre les maths derrière le bail. On tape 3 commandes, ça marche, on saisit plus jamais de password et on a un meilleur niveau de sécurité. La vie est belle.

L'idée c'est :

- une seule fois, vous générez une paire de clés sur VOTRE PC
  - votre clé privée, vous l'ouvrez jamais, ne la regardez jamais, ne la partagez jamais, jamais quoi
  - la clé publique on s'en balec, tu peux la donner partout, donc c'est celle-ci qu'on partage
- vous déposez la clé publique sur la machine à laquelle vous souhaitez vous connecter
  - ou sur les 10000 machines auxquelles vous voulez vous connecter
- vous pouvez désormais vous connecter sans saisir de password avec un meilleur niveau de sécu

Nice and easy. 

### B. Manips

Let's go :

🌞 **Générer une paire de clés**

```stanislasthabault@MacBook-Pro-de-Stanislas ~ % ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/stanislasthabault/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/stanislasthabault/.ssh/id_rsa
Your public key has been saved in /Users/stanislasthabault/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:99KU0q5Apl8GrGF5+AfZ05PSvE894qp3hYL1slNPs2g stanislasthabault@MacBook-Pro-de-Stanislas.local
The key's randomart image is:
+---[RSA 4096]----+
|                 |
|                 |
|                 |
|       + o * o   |
|      = S O @ .  |
|     . O = @ * = |
|      o o = X.*.+|
|       . = *.E.o.|
|        ..+.=..  |
+----[SHA256]-----+
```

```
stanislasthabault@MacBook-Pro-de-Stanislas .ssh % ls
id_rsa		id_rsa.pub	known_hosts	known_hosts.old
stanislasthabault@MacBook-Pro-de-Stanislas .ssh % 
```
🌞 **Déposer la clé publique sur une VM**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % ssh-copy-id -p 22000 et0@10.7.1.254  
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/stanislasthabault/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
et0@10.7.1.254's password: 

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh -p 22000 'et0@10.7.1.254'"
and check to make sure that only the key(s) you wanted were added.
```

🌞 **Connectez-vous en SSH à la machine**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % ssh -p 22000 et0@10.7.1.254 
Last login: Sun Dec  3 23:33:06 2023 from 10.7.1.1
[et0@routeur ~]$ 
```

### C. Changement de fingerprint

Si un jour, quand tu te connectes à un serveur auquel tu t'es déjà connecté avant (genre sur la même adresse IP) mais que le fingerprint a changé, alors la connexion va échouer.

En effet comme on a dit, grâce au fichier `known_hosts` votre PC enregistre les fingerprint que vous acceptez et compare automatiquement à chaque fois.

Vous mangez donc un beau message d'avertissement et la connexion échoue si ça arrive.

On va provoquer cette situation, pour que vous expérimentiez le truc. Ce qu'on va faire dans cette partie :

- supprimer les clés sur le serveur SSH
- regénérez des nouvelles clés (les empreintes seront donc différentes)
- faire une connexion SSH et manger un beau message d'avertissement avec une connexion échouée
- remédier à la situation

🌞 **Supprimer les clés sur la machine `router.tp7.b1`**

```
[et0@routeur ssh]$ ls
moduli        ssh_host_ecdsa_key      ssh_host_ed25519_key.pub  sshd_config
ssh_config    ssh_host_ecdsa_key.pub  ssh_host_rsa_key          sshd_config.d
ssh_config.d  ssh_host_ed25519_key    ssh_host_rsa_key.pub
[et0@routeur ssh]$ sudo rm ssh_host*
[et0@routeur ssh]$ ls
moduli  ssh_config  ssh_config.d  sshd_config  sshd_config.d
```

🌞 **Regénérez les clés sur la machine `router.tp7.b1`**

```
[et0@routeur ssh]$ sudo ssh-keygen -A
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519 
[et0@routeur ssh]$ sudo systemctl restart sshd
[et0@routeur ssh]$ ls
moduli        ssh_host_dsa_key      ssh_host_ecdsa_key.pub    ssh_host_rsa_key      sshd_config.d
ssh_config    ssh_host_dsa_key.pub  ssh_host_ed25519_key      ssh_host_rsa_key.pub
ssh_config.d  ssh_host_ecdsa_key    ssh_host_ed25519_key.pub  sshd_config
```

🌞 **Tentez une nouvelle connexion au serveur**

```
stanislasthabault@MacBook-Pro-de-Stanislas ~ % ssh -p 22000 et0@10.7.1.254                      
The authenticity of host '[10.7.1.254]:22000 ([10.7.1.254]:22000)' can't be established.
ED25519 key fingerprint is SHA256:i32JhWtT0/cZBK2zVAgKCpB2wFWMSl/TLpNTQHzRcIE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.7.1.254]:22000' (ED25519) to the list of known hosts.
Last login: Sun Dec  3 23:47:29 2023 from 10.7.1.1
[et0@routeur ~]$ 
```


# III. Web sécurisé

On va mettre en place un petit serveur HTTPS dans cette section. On va encore héberger un MEOW, MAIS de façon sécurisée.

- [III. Web sécurisé](#iii-web-sécurisé)
  - [0. Setup](#0-setup)
  - [1. Setup HTTPS](#1-setup-https)

![HTTPS](./img/do_you_even.jpg)

## 0. Setup

| Name            | LAN1 `10.7.1.0/24` |
| --------------- | ------------------ |
| `router.tp7.b1` | `10.7.1.254`       |
| `john.tp7.b1`   | `10.7.1.11`        |
| `web.tp7.b1`    | `10.7.1.12`        |

➜ **Créez la machine `web.tp7.b1`** et déroulez la checklist.

➜ **Référez-vous au TP5 et déroulez la partie II.3. Serveur Web sur `web.tp7.b1`**

- ne passez à la suite qu'une fois que votre serveur web est disponible en HTTP
- faites les tests depuis `john` avec la commande `curl` ou depuis votre PC avec un navigateur ou la commande `curl`

🌞 **Montrer sur quel port est disponible le serveur web**

- avec une commande `ss` sur la machine `web.tp7.b1`

## 1. Setup HTTPS

Pour avoir un beau cadenas vert à côté de la barre d'URL dans le navigateur, il faut avoir une IP publique, un nom de domaine public etc, et this costs money sir.

Dans notre lab ici, on va fabriquer un certificat "auto-signé". La connexion sera chiffrée, mais cadenas rouge. Sécurisé si vous pouvez attester vous-mêmes de la validité du certificat.

🌞 **Générer une clé et un certificat sur `web.tp7.b1`**

```bash
# génération de la clé et du certificat
$ openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt

# on déplace la clé dans un répertoire standard pour les clés
# et on la renomme au passage
$ sudo mv server.key /etc/pki/tls/private/web.tp7.b1.key

# pareil pour le cert
$ sudo mv server.crt /etc/pki/tls/certs/web.tp7.b1.crt

# on définit des permissions restrictives sur les deux fichiers
$ sudo chown nginx:nginx /etc/pki/tls/private/web.tp7.b1.key
$ sudo chown nginx:nginx /etc/pki/tls/certs/web.tp7.b1.crt
$ sudo chmod 0400 /etc/pki/tls/private/web.tp7.b1.key
$ sudo chmod 0444 /etc/pki/tls/certs/web.tp7.b1.crt
```

🌞 **Modification de la conf de NGINX**

- je vous montre le fichier avec `cat`, éditez-le avec `nano` de votre côté

```bash
[it4@web ~]$ sudo cat /etc/nginx/conf.d/site_web_nul.conf
server {
    # on change la ligne listen
    listen 10.7.1.12:443 ssl;

    # et on ajoute deux nouvelles lignes
    ssl_certificate /etc/pki/tls/certs/web.tp7.b1.crt;
    ssl_certificate_key /etc/pki/tls/private/web.tp7.b1.key;

    server_name www.site_web_nul.b1;
    root /var/www/site_web_nul;
}
```

🌞 **Conf firewall**

- ouvrez le port 443/tcp dans le firewall de `web.tp7.b1`

🌞 **Redémarrez NGINX**

- avec un `sudo systemctl restart nginx`

🌞 **Prouvez que NGINX écoute sur le port 443/tcp**

- avec une commande `ss`

🌞 **Visitez le site web en https**

- avec un `curl` depuis `john.tp7.b1`
  -  il faudra ajouter l'option `-k` pour que ça marche : `curl -k https://10.7.1.12`
- et testez avec votre navigateur aussi, histoire de !