+++
author = "remy"
categories = ["linux"]
date = "2014-10-31"
description = "Tutoriel sur l'installation et la configuration d'un serveur VPN sous Debian grâce à OpenVPN."
featured = "openvpn-logo.png"
featuredalt = "Logo OpenVPN"
featuredpath = "/img/posts/configurer-serveur-vpn-openvpn"
linktitle = ""
title = "Configurer un serveur VPN avec OpenVPN"
type = "post"

+++


Vous souhaitez un peu plus d’anonymat sur internet, surpasser le blocage d’un serveur proxy ou simplement créer un réseau local entre des postes géographiquement éloignés ? Voici votre solution : un serveur VPN.

Dans ce tutoriel, nous allons apprendre à configurer un serveur VPN et à s’y connecter depuis Windows ou Linux.
<!--more-->

## OpenVPN, késako ?

[OpenVPN][openvpn] est un logiciel lancé par James Yonan permettant de créer un réseau privé virtuel (VPN en anglais). Le protocole OpenVPN est l’un des plus utilisé pour les serveurs VPN (open-source et multi-plateforme).

Si vous souhaitez vous renseigner plus sur le fonctionnement d’un VPN, voici un article de Comment ça marche ? : [« VPN – Réseaux Privés Virtuels »][ccmvpn].


## 1. Installation

Sans plus attendre, passons à l’installation :

Comme vous devez vous en douter, il va nous falloir installer OpenVPN qui se trouve sur les dépôts. Mais en plus d’OpenVPN, nous allons avoir besoin d’OpenSSL qui nous permettra de générer les certificats essentiels à l’utilisation du VPN.
```shell
root@tuto:~# apt-get install openvpn openssl
```
Nous allons de suite générer les certificats utiles au serveur. OpenVPN nous offre dans sa documentation (installée en même temps qu’OpenVPN) des scripts permettant de générer facilement les certificats (que ce soit pour le serveur ou les clients). Nous allons donc copier ces scripts dans notre dossier OpenVPN :

```shell
root@tuto:~# cp -R /usr/share/doc/openvpn/examples/easy-rsa/2.0/ /etc/openvpn/easy-rsa/
root@tuto:~# cd /etc/openvpn/easy-rsa/
```

Sous Debian 8 (Jessie), utiliser */usr/share/easy-rsa/* à la place de */usr/share/doc/openvpn/examples/easy-rsa/2.0/*

Les scripts easy-rsa ont besoin de variables d’environnement spécifiques. Elles sont toutes contenues dans le fichier vars et déjà préconfigurées, mais vous pouvez les modifier à votre guise. Nous allons donc charger ce fichier :
```shell
root@tuto:/etc/openvpn/easy-rsa# source vars
```
Une note apparait normalement sur la console vous disant que l’exécution de la commande `./clean-all` supprimera tous les certificats. Par précaution, on va l’exécuter, pour avoir une installation propre :
```shell
root@tuto:/etc/openvpn/easy-rsa# ./clean-all
```
Nous pouvons maintenant commencer à générer tous les certificats / clés qui seront utilisés par le serveur VPN :

1. Le premier certificat à générer est celui de l’autorité de certification qui permettra de vérifier que les certificats proviennent bien de notre serveur : `./build-ca`

2. Ensuite, nous allons générer le certificat du serveur OpenVPN (nommé ici VPN) : `./build-key-server VPN`
Lors de la génération de ce certificat, ne mettez surtout pas de mot de passe : il est utilisé par le serveur, il sera donc inutilisable avec un mot de passe.
3. Nous allons ensuite générer la clé Diffie-Hellman permettant l’échange de certificats, entre le serveur et le client avant la connexion, de manière sécurisée : `./build-dh`
4. Pour augmenter la sécurité de la connexion, nous allons obliger les clients à se connecter avec une clé d’authentification TLS : `openvpn --genkey --secret keys/ta.key`
5. Pour en finir avec les certificats (pour le moment), nous allons générer le certificat qui contiendra les certificats révoqués (crl.pem) : `./revoke-full client`

Le script `./revoke-full` permet de révoquer un certificat. Mais lors de la première exécution, comme le certificat crl.pem n’existe pas, il sera créé.  
En exécutant ce script, vous devriez avoir une erreur qui vous dit (vulgairement) que le certificat client n’existe pas. C’est tout a fait normal étant donnée que l’on ne veut pas le révoquer, seulement créer la base des certificats révoqués.

Nous en avons maintenant terminé avec les certificats, nous pouvons nous attaquer à la configuration.

Si vous ne voulez pas trop vous prendre le choux avec la configuration, voici ma configuration. Sinon, L’exemple de documentation est disponible ici : */usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz*  
La configuration est a placer dans le fichier */etc/openvpn/server.conf*

```
mode server
# Définition du type de liaison (ici, c'est un serveur routé)
dev tun
# Protocole utilisé
proto tcp
# Port utilisé
port 443
# Utilise un sous-réseau global pour le VPN (par défaut : un sous-réseau par clients) + d'infos : https://goo.gl/PJMFOW
topology subnet
# Certificats
# Autorité de certification
ca /etc/openvpn/easy-rsa/keys/ca.crt
# Certificat serveur
cert /etc/openvpn/easy-rsa/keys/VPN.crt
# Clé serveur
key /etc/openvpn/easy-rsa/keys/VPN.key
# Certificat DH (dh2048.pem pour Débian 8)
dh /etc/openvpn/easy-rsa/keys/dh1024.pem
# Liste des certificats révoqués
crl-verify /etc/openvpn/easy-rsa/keys/crl.pem
# Clé TLS de chiffrement et d'authentification de la connexion
tls-crypt /etc/openvpn/easy-rsa/keys/ta.key

#Plage d'ip et masque réseau
server 10.8.0.0 255.255.255.0
persist-key
persist-tun
#Chemin du fichier qui contiendra le statut de chaque connexion active
status /var/log/openvpn-status.log
#Niveau de verbose
verb 1
# Autorise les clients a "voir" les autres clients
client-to-client
push "redirect-gateway bypass-dhcp"
# Définition des DNS qui seront utilisés par le client (ici DNS de Google
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
# Chemin des logs
log-append /var/log/openvpn.log
# Activation de la compression
comp-lzo
# Définition du type de chiffrage
cipher AES-128-CBC
# Activation des ip statiques des clients (stockées dans le fichier ipp.txt)
ifconfig-pool-persist ipp.txt 1
# Options de maintient de connexion (1 ping toutes les 10s et déconnexion si plus de contact pendant 120s)
keepalive 10 120
```

Le serveur est maintenant correctement configuré, on peut donc le redémarrer :

```shell
 root@tuto:/etc/openvpn/easy-rsa# service openvpn restart
```

Si tout c’est correctement déroulé, votre serveur est maintenant fonctionnel. Mais si un client se connecte à ce stade, il n’aura pas accès à internet. Il faut donc que le serveur fasse office de routeur entre l’interface réseau du VPN (tun0) et l’interface ayant accès à internet (par défaut eth0).

La première chose à faire est d’activer « l’ip forward » :
```shell
root@tuto:/etc/openvpn/easy-rsa# echo 1 > /proc/sys/net/ipv4/ip_forward
```

Mais cette commande ne permet pas de garder le paramètre activé après un redémarrage. Il faut donc modifier le fichier */etc/sysctl.conf* et décommentez cette ligne (ou ajoutez la si elle n’est pas présente) :

```
net.ipv4.ip_forward = 1
```

Il faut ensuite régler iptables pour qu’il redirige le trafic. Personnellement, j’utilise un script se lançant au démarrage.

1. Créez et éditez le script */etc/init.d/firewall* et insérez-y le script ci-dessous :

    ```bash
    ### BEGIN INIT INFO
    # Provides: Firewall
    # Required-Start: $local_fs $network
    # Required-Stop: $local_fs $remote_fs
    # Default-Start: 2 3 4 5
    # Default-Stop: 0 1 6
    # Short-Description: Firewall
    # Description: Activate Firewall
    ### END INIT INFO

    #!/bin/sh
    # Autoriser le routage depuis et vers l'interface réseau du VPN :
    iptables -I FORWARD -i tun0 -j ACCEPT
    iptables -I FORWARD -o tun0 -j ACCEPT
    iptables -I OUTPUT -o tun0 -j ACCEPT

    # Translation d'adresses
    iptables -A FORWARD -i tun0 -o eth0 -j ACCEPT
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
    ```

2. Rendez le script exécutable en modifiant ses droits d’accès :

    ```shell
    root@tuto:/etc/openvpn/easy-rsa# chmod +x /etc/init.d/firewall
    ```
3. Maintenant que le script est prêt, il faut l’ajouter aux scripts à exécuter lors du démarrage :

    ```shell
    root@tuto:/etc/openvpn/easy-rsa# update-rc.d firewall defaults
    ```
4. On exécute le script (pour éviter de redémarrer la machine) :

    ```shell
    root@tuto:/etc/openvpn/easy-rsa# /etc/init.d/firewall
    ```

Le serveur est maintenant totalement configuré pour accueillir les connexions des clients.


## 2. Création d’un certificat pour les clients

Je vais vous proposer deux méthodes pour créer les certificats/configurations des clients. La première utilise un script permettant de générer les configurations en une seule commande. Le second est la création manuelle.

Cette opération sera à renouveler pour chaque client.

### 1<sup>ère</sup> méthode :

Je vous invite à vous rendre sur cette page pour suivre les instructions d’installation du script : [https://gitlab.com/remyj38/Creation-de-client-openvpn/][scriptCreationConfig]

Il ne vous reste plus qu’à exécuter le script avec les paramètres demandés :
```shell
root@tuto~# ./créer_client.sh toto toto@domaine.com
```
Et c’est tout, la configuration a été envoyée par mail à l’adresse donnée.

### 2<sup>nde</sup> méthode :

Nous allons retourner dans le dossier easy-rsa pour générer les certificats client :
```shell
root@tuto:~# cd /etc/openvpn/easy-rsa/ root@tuto:/etc/openvpn/easy-rsa# source vars
```
Le script *./build-key* permet de générer facilement les certificats d’un client. Nous n’allons pas nous en priver 😀
```shell
root@tuto:/etc/openvpn/easy-rsa# ./build-key nom_du_client
```
Reste maintenant à créer la configuration du client. Idem que pour la configuration serveur, voici ma configuration si vous ne souhaitez pas utiliser celle par défaut :
```
client
#Mode routé
dev tun
#Protocole
proto tcp-client
# Ip du serveur
remote ip_serveur 443
resolv-retry infinite
nobind
persist-key
persist-tun
key-direction 1
cipher AES-128-CBC
ns-cert-type server
redirect-gateway
route-delay 2
comp-lzo
verb 1
#Certificats et clés
ca ca.crt
cert nom_du_client.crt
key nom_du_client.key
tls-crypt ta.key
```

Dans cette configuration, il vous faut modifier l’ip du serveur (`ip_serveur`), et le nom du client pour le cert et le key.

*Ce fichier est à nommer en .ovpn*

Récupérez ensuite ces fichiers sur votre serveur : *ca.crt*, *nom_du_client.crt*, *nom_du_client.key* et *ta.key*. Lors de l’installation de la configuration, il vous faudra les placer dans le dossier de configuration.

## 3. Installation client

A ce stade du tutoriel, nous avons le serveur qui est fonctionnel et notre configuration client sous la main. Il nous reste puisqu’à installer le client OpenVPN ce sera fini.

### Installation sous Windows

Pour Windows, il vous faudra télécharger l’installer sur le site d’OpenVPN : [http://openvpn.net/index.php/open-source/downloads.html][openvpnDownload]

Lors de l’installation, laissez les choix par défaut pour le choix des options.

Lorsque l’installation est terminée, rendez-vous dans le dossier de configurations d’OpenVPN (lien présent dans le menu démarrer, sinon dans C:\Program Files\OpenVPN\Config).

Mettez-y votre configuration (et potentiellement vos certificats si vous avez utilisé la 2<sup>nde</sup> méthode).

Lancez ensuite OpenVPN et connectez-vous au serveur (l’icône est apparu dans la barre de notifications). Mais attention, OpenVPN doit toujours être lancé en tant qu’administrateur.

### Installation sous Linux (Ligne de commande)

Installez le paquet openvpn :
```shell
root@tuto:~# apt-get install openvpn
```
Placez ensuite la configuration dans le dossier */etc/openvpn* sous le nom de *client.conf*

OpenVPN se lance et se connecte automatiquement au démarrage de la machine.

Et voila, le serveur VPN est fonctionnel et les clients peuvent se connecter dessus.

[ccmvpn]: http://www.commentcamarche.net/contents/514-vpn-reseaux-prives-virtuels-rpv "Détail sur le fonctionnement d'un VPN"
[openvpn]: http://www.openvpn.net "Site web d'OpenVPN"
[openvpnDownload]: http://openvpn.net/index.php/open-source/downloads.html
[scriptCreationConfig]: https://gitlab.com/remyj38/Creation-de-client-openvpn/tree/master
