+++
author = "remy"
categories = ["linux", "web"]
date = "2016-11-01"
description = "Dans ce tutoriel, vous allez apprendre à utiliser Let's Encrypt afin d'utiliser le protocole HTTPS sur un serveur Apache"
featured = "webserver.png"
featuredalt = "Bannière de l'article"
featuredpath = "/img/posts/activer-https-apache-lets-encrypt"
linktitle = ""
title = "Activer le HTTPS sur Apache avec Let's Encrypt"
type = "post"

+++

Si vous souhaitez sécuriser les flux de données entre votre serveur web et les clients, afin de protéger des données pouvant être sensible (comme des identifiants de connexion et autres)  ou tout autre raison, vous allez devoir utiliser le protocole HTTPS.  Qu’est-ce que le HTTPS ? C’est tout simplement le protocole HTTP, avec une couche de chiffrement (SSL ou TLS) afin de le sécuriser, via un chiffrement asymétrique, l’échange de données entre le client et le serveur. Nous allons donc voir comment générer un certificat Let’s Encrypt et comment configurer Apache pour qu’il active correctement HTTPS.

<!--more-->

# 1. Petit point théorique

Avant d’entrer dans la technique, nous allons clarifier un peu le fonctionnement du SSL/TLS. Pas d’inquiétudes, on ne va pas rentrer dans les détails (si vous voulez vous documenter, internet regorge de très bons articles à ce sujet). Nous allons simplement voir un point qui est important pour la suite : les certificats.


## Le SSL/TLS

Le SSL/TLS (j’utiliserais seulement le terme SSL par la suite) est basé sur un système de pairs de clés : le serveur utilise une paire de clé (publique et privée) afin de sécuriser la connexion. Afin de certifier que la paire de clé utilisée est bien celle du bon serveur, nous utilisons un système ce certificats. Il contient plusieurs informations afin de permettre au client de savoir avec qui il communique. Si l’on s’arrête ici, un navigateur qui reçoit un certificat n’a aucun moyen de savoir si le certificat est authentique ou non (tout le monde peu générer des certificats). Internet se base donc sur des chaines de confiance : le certificat utilisé par le site X est signé par une autorité de certification. L’autorité de certification certifie donc que le certificat est bien authentique.

Toutes les autorités de certifications sont directement incluses dans la configuration du navigateur (et du système d’exploitation). Par exemple, dans Firefox, si vous vous rendez dans les paramètres, onglet « Avancé » puis « Certificats », vous y trouverez la liste de toutes les autorités de certifications reconnues par Mozilla.

![Liste des autorités de certification reconnues par Mozilla][image1]  
*Liste des autorités de certification reconnues par Mozilla*

Donc, si notre serveur web n’utilise pas de certificat signé par une des autorités reconnues par tous les navigateurs, le navigateur affichera un beau message au visiteur comme quoi la connexion n’est pas sûre (de quoi le décourager de venir sur votre site).

![Connexion à un site avec un certificat non valide][image2]
*Connexion à un site avec un certificat non valide*

Jusqu’à il n’y a pas longtemps, avoir un «vrai» certificat coutait chère (d’une 50aine d’euros à plusieurs milliers) car les autorités de certifications offrent diverses options avec le certificat (support, garantie, …). Mais dans le cas de petits hébergements, nous n’avons pas besoin d’avoir tous ces services. Nous allons donc utiliser Let’s Encrypt, qui est gratuit.


## Let’s Encrypt

Let’s Encrypt a été lancé il y a presque un an de cela par l’Internet Security Research Group (ISRG) et a très rapidement été rejoint pas des grands acteurs du web ([Mozilla, l’EFF, Akamai, Cisco, Facebook, …][LetsEncryptSponsors]). Le but du projet est d’avoir un web 100% chiffré pour des raisons de sécurité et de vie privée.

Le projet a très rapidement pris une ampleur démesurée puisque les grands hébergeurs web proposent maintenant le HTTPS gratuitement à tous leurs clients grâce à Let’s Encrypt. Cela à permis de dépasser la barre des 10 millions de certificats actifs fournis par Let’s Encrypt :

![Évolutions des certificats fournis par Let's Encrypt][image3]  
*Évolutions des certificats fournis par Let’s Encrypt*

### Procédure de certification de Let’s Encrypt

Contrairement aux autres autorités de certifications, Let’s Encrypt a été pensé pour faciliter la tâche des administrateurs : tout est automatique. Le processus de validation est donc différents de celui utilisé par les autres (qui demandent de valider le domaine par mail avant de pouvoir émettre des certificats). Voici donc le procédé (en simplifié) :

1. Le serveur voulant signer un domaine (exemple.com pour l’exemple) va demander faire la demande à Let’s Encrypt. Afin de vérifier que le serveur demandant le certificat pour le domaine demandé est authentique, Let’s Encrypt demande au serveur d’ajouter un fichier sur le site correspondant au domaine.

    ![Vérification du domaine][image4]  
    *Vérification du domaine*

2. Suite à ça, le serveur peut transmettre la demande de certificat signée avec une clé privée à Let’s Encrypt. Si Let’s Encrypt arrive à atteindre le fichier précédemment ajouté, Let’s Encrypt va signer renvoyer le certificat signé au serveur.
    ![Signature du certificat][image5]  
    *Signature du certificat*

3. Il ne reste plus qu’à configurer le certificat pour que tout fonctionne.

# 2. Générer un certificat avec Let’s Encrypt

La procédure de génération d’un certificat est totalement automatique avec Let’s Encrypt : pas besoin de générer une clé, de faire la demande de certificat pour la transmettre à l’autorité de certification.

Pour la suite du tutoriel, j’utiliserais cette configuration :

- Système d’exploitation : *Debian 8 Jessie*
- Serveur web : *Apache* ([tutoriel d’installation][tutorielLAMP]),
- FQDN : *tutoriel.initmanfs.eu* et *tutoriel2.initmanfs.eu* ([configuré comme un seul vhost][tutorielVhosts])
- Dossier racine de l’hébergement web : */var/www/tutoriel.initmanfs.eu*/


## Installation de Certbot

Il existe beaucoup de logiciels pour générer des certificats avec Let’s Encrypt. De notre coté, nous utiliserons le client officiel : [Certbot][Certbot]. Debian ayant une procédure de mise à jour des paquets très lente (due aux nombreuses validations avant la publication), nous allons devoirs piocher dans les dépôts de test (pas d’inquiétude, seul Certbot et ces dépendances utiliserons les dépôts de test). Pour ce faire, ajoutez la ligne suivante à votre fichier */etc/apt/sources.list* :

```
deb http://ftp.debian.org/debian jessie-backports main
```

Ensuite, nous allons utiliser `apt-get` pour installer les paquets requis :

```shell
remy@tuto:~$ sudo apt-get update && sudo apt-get install python-certbot-apache -t jessie-backports
```

## Génération du certificat

Certbot étant maintenant installé, il ne nous reste plus qu’à nous lancer pour valider les domaines.

Petite précision avant de nous lancer : je préfère avoir un certificat par site plutôt qu’un certificat pour l’ensemble de mes site. Je m’explique : Afin qu’un visiteur ne puisse pas avoir la liste de tous mes sites hébergés sur mon serveur, je génère un certificat par vhost. Deuxième précision : je n’aime pas les fichiers de configuration générés par le bot ni ses paramètres par défaut, j’utilise donc l’option de génération seule de certificat.

Parenthèse close, nous allons générer le certificat pour les domaines *tutoriel.initmanfs.eu* et *tutoriel2.initmanfs.eu*. Rien de plus simple :

```shell
remy@tuto:~$ sudo certbot --apache certonly --rsa-key-size 4096 -d tutoriel.initmanfs.eu -d tutoriel2.initmanfs.eu
```

Explications :

- *--apache* : On demande à certbot de regarder dans la configuration d’Apache pour récupérer les domaines et leur configuration
- *certonly* : Ce paramètre permet de dire que l’on génère seulement le certificat, sans l’installer
- *--rsa_key_size 4096* : Par défaut, certbot génère des clés de 2048 bits. Je préfère utiliser des clés de 4096 bits.
- *-d tutoriel.initmanfs.eu* : Cela permet de définir les domaines à valider dans le certificat. Vous pouvez en ajouter autant que vous le souhaitez. Vous pouvez également omettre ce paramètre, une interface graphique s’ouvrira alors pour vous demander de les sélectionner.

Après avoir lancé la commande, Certbot va vous demander deux informations : l’adresse mail de contact (en cas de perte des identifiants qui seront générés) et l’acceptation des conditions générales d’utilisation.  
 Lorsque la génération est terminée, il vous indique plusieurs choses :

- La première comme quoi votre certificat est généré et présent à l’emplacement donné (*/etc/letsencrypt/live/tutoriel.initmanfs.eu/fullchain.pem* pour ma part).
- Seconde chose à retenir : Il vous dit qu’il faut que vous sauvegardiez les informations présentes dans le dossier /etc/letsencrypt. En effet, ce dossier contient les certificats et clés générés, mais également les informations permettant de générer les certificats pour le domaine. Donc très important de le sauvegarder régulièrement.


## Configuration du renouvellement automatique

Les certificats Let’s Encrypt n’étant valide que 3 mois, on ne va pas s’amuser à les renouveler à la main. Heureusement, Certbot inclut l’option de renouvellement automatique. Pour cela, nous allons ajouter la commande en tache cron (`sudo crontab -e` pour rappel) :

```
0 */12 * * * certbot renew -q --rsa-key-size 4096 --post-hook "systemctl reload apache"
```

Nous exécutons donc le renouvellement deux fois par jours, en mode silencieux (-q). Pourquoi deux fois par jour me diriez-vous ? Car c’est la recommandation de Certbot : en cas d’échec, cela nous permet d’avoir un deuxième essai.

# 3. Installation du certificat

La toute première chose à faire est d’activer le module SSL d’Apache ainsi que le module Headers avec la commande `a2enmod ssl headers`. Ensuite, pour installer le certificat, nous reprendrons la configuration présente dans le tutoriel sur [hôtes virtuels][tutorielVhosts] en modifiant quelques parties :

- Première chose à faire : modifier le port du vhost. En effet, le port par défaut du HTTPS est le 443.
- Ajouter les deux lignes suivantes dans la configuration du vhost :

```apache
SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/tutoriel.initmanfs.eu/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/tutoriel.initmanfs.eu/privkey.pem
Header always set Strict-Transport-Security "max-age=15768000"
```

La première ligne active le SSL sur le vhost. Les deux lignes suivantes deviennent donc obligatoires. La seconde ligne permet de donner le chemin du certificat (chemin donné par certbot). La troisième quand à elle définie le chemin de la clé privée.  
Pour ce qui est de la dernière, permet de dire au navigateur de toujours utiliser une connexion sécurisée (donc avec un certificat valide) pour ce connecter au serveur.

Il ne reste plus qu’à redémarrer Apache et vous pourrez accéder à votre site sans aucun problème.


## Sécuriser le module SSL

Plusieurs protocoles SSL et une quantité de ciphers sont aujourd’hui considérés comme non sûr. Afin de palier à ce problème, nous allons les désactiver. Dieu merci, Mozilla propose un outil pour générer la configuration : [https://mozilla.github.io/server-side-tls/ssl-config-generator/][sslGenerator]  
Il vous suffit d’entrer la version d’Apache (`apache2 -v`) et d’openSSL (`openssl version`) et vous avez votre configuration.  
Ajoutez les directives `SSLProtocol`, `SSLCipherSuite`, `SSLHonorCipherOrder` et `SSLCompression` à la fin de votre fichier */etc/apache2/mods-available/ssl.conf* (avant le `</Ifmodule>`).

Rechargez la configuration d’Apache et le tour est joué. Vous pouvez tester votre site sur [SSL Labs][ssltest]. Si vous avez bien suivi ce tutoriel, vous devriez avoir une notre de A+ :

![Rang A+ SSL Labs][image6]
*Rang A+ SSL Labs*


## Rediriger tout le trafic vers le HTTPS

Si vous souhaitez que votre site soit accessible seulement en HTTPS, il vous suffit de rajouter la ligne suivante dans la configuration du vhost HTTP :

```apache
Redirect Permanent / https://tutoriel.initmanfs.eu/
```
[Ma configuration finale][finalConfig]

Et voila, vous avez un serveur acceptant les requêtes en HTTPS, avec un certificat valide (qui ne fera pas fuir les visiteurs). Ça ne faisait pas parti du tutoriel donc je ne l’ai pas cité, mais faites attentions avec le contenu mixte (fait d’avoir des ressources chargée par une page en non sécurisé). Cela peu facilement casser un site 😉

# 4. Liens Utiles :

- [Documentation de Certbot][certbotDoc]
- [Documentation Apache sur le module SSL][apache2SSL]
- [Le générateur de configuration SSL de Mozilla][sslGenerator]
- [Documentation du MDN sur le contenu mixte (en anglais)][mixedContent]

[apache2SSL]: https://httpd.apache.org/docs/current/mod/mod_ssl.html "Documentation Apache sur le module SSL"
[certbot]: https://certbot.eff.org/ "Site de Certbot"
[certbotDoc]: https://certbot.eff.org/docs/using.html "Documentation de Certbot"
[finalConfig]: /uploads/tutoriel.initmanfs.eu.conf
[LetsEncryptSponsors]: https://letsencrypt.org/sponsors/ "Sponsors de Let's Encrypt"
[mixedContent]: https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content "Documentation du MDN sur le contenu mixte (en anglais)"
[sslGenerator]: https://mozilla.github.io/server-side-tls/ssl-config-generator/ "Générateur de configuration SSL de Mozilla"
[sslTest]: https://www.ssllabs.com/ssltest "Test SSL de SSl Labs"
[tutorielLAMP]: https://blog.remyj.fr/linux/serveur-web-apache-php-mysql/ "Tutoriel d'installation d'un serveur web"
[tutorielVhosts]: https://blog.remyj.fr/linux/ajouter-des-hotes-virtuels-sur-un-serveur-apache/ "Tutoriel de configuration des hôtes virtuels"

[image1]: /img/posts/activer-https-apache-lets-encrypt/ca-mozilla.png "Liste des autorités de certification reconnues par Mozilla"
[image2]: /img/posts/activer-https-apache-lets-encrypt/connexion-non-securisee.png "Connexion à un site avec un certificat non valide"
[image3]: /img/posts/activer-https-apache-lets-encrypt/evolution-lets-encrypt.png "Évolutions des certificats fournis par Let's Encrypt"
[image4]: /img/posts/activer-https-apache-lets-encrypt/lets-encrypt-howitworks_challenge.png "Vérification du domaine"
[image5]: /img/posts/activer-https-apache-lets-encrypt/lets-encrypt-howitworks_authorization.png "Signature du certificat"
[image6]: /img/posts/activer-https-apache-lets-encrypt/test-ssl.png "Rang A+ SSL Labs"
