---
title: "GOGS, un github version light en auto-hébergement "
description: "Installation et avantages de cette solution"
date: 2020-01-31T20:21:20+01:00
publishDate: 2020-01-31T20:21:20+01:00
author: "Derios"
images: []
draft: false
tags: ["gogs", "git","auto-hébergement"]
---

# Présentation de la solution

C'est  une solution light de github, qu'on peut utiliser "presque" partout et rapide.

De plus, les utilisateurs de github ne seront pas perdu, c'est la mm chose avec des fonctionnalitées en moins (pas de ci).

Enfin, pas de CI... intégré du moins, il s'intègre parfaitement à un jenkins.

 Je l'ai utilisé et maintenu un an en production sans avoir eu à me préoccuper de sa maintenance hormis la classque install, configuration, politique de backup et PRA classique.

## Compatibilité

Version 0.11.91@2019-08-11

- linux : 386, amd64, arm
- windows : 386, amd64
- macOS : amd64

# Installation

## Prérequis

J'ai choisi comme frontal apache 2.4 (pour changer un peu) mais il y a bien d'autres solutions (notamment Nginx, traefik, istio,envoy suivant l'infrastructure).

Une distribution linux, en l'occurence une vm CentOs 7.

Gogs prend en charge de nombreuses base de données.

## l'install

### Création d'un utilisateur.

```bash
su - root
useradd -m -s /bin/bash git # ajout de l utilisateur
passwd git # création du mot de passe
```

Pour les mots de passes, par expérience on évite les caractères spéciaux, l'idéal étant de passer par un utilitaire de type keepass et de générer du 128 voir 256 bit pour la prod.

### Téléchargement de gogs

[librairie officielle](https://gogs.io/docs/installation/install_from_binary)

```bash
cd /home/git
yum install wget unzip vim
wget https://dl.gogs.io/0.11.91/gogs_0.11.91_linux_amd64.zip && chown git: gogs_0.11.91_linux_amd64.zip
su - git
unzip gogs_0.11.91_linux_amd64.zip && rm gogs_0.11.91_linux_amd64.zip
cd gogs
```

### On monte le service

```bash
su - root
cp /home/git/gogs/scripts/systemd/gogs.service /lib/systemd/system
cat /lib/systemd/system/gogs.service # on voit que le path de ExecStart n'est pas le bon
vim /lib/systemd/system/gogs.service
```

```BASH
# Dans mon cas je remplace ExecStart
WorkingDirectory=/home/git/gogs
ExecStart=/home/git/gogs web
Restart=always
```

```bash
systemctl start gogs
systemctl enable gogs
systemctl status gogs
```

On voit bien qu'il tourne sur du http://0.0.0.0:3000.

Plusieurs choix s'offre à nous pour diffuser le service.... bref on lui colle un frontal.

### Donc le frontal

#### Installation

```bash
# AH oui
yum update
# Voila c'est mieux
yum install httpd  mod_ssl #serveur apache et son module qui implémente le ssl
```

#### Configuration

```BASH
vim /etc/httpd/conf.d/gogs.conf
```

Le server name est le  FQDN auquel répond votre virtualhost. ([doc officielle](https://httpd.apache.org/docs/2.4/fr/vhosts/examples.html))

```XML
 <VirtualHost *:80>
    ServerName www.gogs.local
    RewriteEngine on
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,QSA,R=permanent]
</VirtualHost>



<VirtualHost *:443>
        ServerName www.gogs.local
        ProxyPass / http://localhost:3000/
        ProxyPassReverse / http://localhost:3000/
        SSLEngine on
        SSLCertificateFile /etc/pki/tls/private/apache-selfsigned.crt
        SSLCertificateKeyFile /etc/pki/tls/private/apache-selfsigned.key

</VirtualHost>
```

##### Génération des certificats.

Ici je génère une paire de certificat auto-signé.

```BASH
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/pki/tls/private/apache-selfsigned.key -out /etc/pki/tls/private/apache-selfsigned.crt
```

Vérification de la configuration

```bash
apachectl configtest
```
Démarrage du serveur

```bash
systemctl enable httpd
ystemctl start httpd
systemctl status httpd
```

Si comme moi vous être dans un lab sans DNS (bind9 voir pfsense..).

A réaliser sur le client (en l'occurence un linux)
```bash
vim /etc/hosts
```
Sur windows, on ouvre un notepad++(ou autre) en tant qu'administrateur
voici le path `C:\windows\system32\drivers\etc\hosts`

et on rajoute cette ligne, qui nous permettra de résoudre.

```bash
192.168.1.106 www.gogs.local
ping www.gogs.local # on vérifie qu'on résoud bien
```

##### Ouverture des ports sur la vm

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --reload
```

Et voila 
![](/posts/images/gogs_install/gogs.PNG)

Il nous reste plus qu'a configurer les différents éléments et vous aurez un git auto-héberger.

Si jamais vous être peu et/ou des petits projet, sur ce gogs à un sqllite fait largement le boulot.

# Dépannage

En cas de souci, les logs (par défault) sont ici.

```bash
tail -f /var/log/httpd/* # en stream
```

Moyen rapide de déterminer si un service écoute

```bash
yum install net-tools
netstat -netpul | grep httpd # permet de voir si le serveur httpd écoute sur le 443/80, si aucune sortie c'est qu'en effet non..
```

Moyen rapide (en tcp) de déterminer si le service est atteignable sur le port (bien évidemment, il faut d'abord voir si il est en écoute)

```bash
# A partie du client donc
telnet hostname|ip 80
telnet hostname|ip 443
```
