+++
author = "arthur"
categories = ["linux"]
date = "2015-04-21"
description = "Tutoriel permettant d'installer la plateforme de monitoring Linux Dash"
featured = "linux-dash.png"
featuredalt = "Bannière article"
featuredpath = "/img/posts/monitoring-avec-linux-dash"
linktitle = ""
title = "Monitoring Linux avec Linux Dash"
type = "post"

+++


Aujourd’hui, nous allons installer [Linux Dash][linuxdash]

## 1. Installation de Linux Dash

Avant d’installer Linux Dash, vous devez avoir installé un serveur web sur votre machine. Suivez ce [tutoriel][tutoLAMP] si vous ne savez pas comment faire.

Dans le répertoire des fichiers web (*/var/www/* par défaut), nous allons télécharger l’archive de linux Dash à partir de Github :

```shell
root@tuto:/var/www# wget https://github.com/afaqurk/linux-dash/archive/master.tar.gz
```

Après avoir téléchargé l’archive, nous pouvons maintenant l’extraire :

```shell
root@tuto:/var/www# tar -xvf master.tar.gz
```

Cette archive ne nous est plus utile, nous pouvons donc la supprimer.

```shell
root@tuto:/var/www# rm master.tar.gz
```

Une fois cette archive extraite, vous pouvez accéder à votre plateforme de monitoring en entrant cette adresse dans votre navigateur:
<http://votreip/linux-dash-master>

## 2. Sécurisation de Linux Dash

Afin de protéger cette page, nous allons mettre en place un fichier htaccess. N’oubliez pas le point devant le fichier. Ce point permet de cacher le fichier.

```shell
root@tuto:/var/www# nano linux-dash-master/.htaccess
```

Ensuite copiez le code ci-dessous, il vous permettra de protéger l’accès au dossier contenant le fichier htaccess par un mot de passe :

```apache
AuthUserFile /var/.htpasswd
AuthName "Acces Restreint"
AuthType Basic
require valid-user
```

Détail des directives :

- AuthUserFile : Permet de définir le chemin vers le fichier contenant les associations logins/mots de passe
- AuthName : Message affiché lors de la demande d’identifiant
- AuthType : Type de l’authentification ([plus d’infos][apacheAuthType])
- Require : Permet de restreindre l’accès à une ressource. Dans notre cas, on restreint l’accès aux utilisateurs ayant entré un bon login.

Il nous faut ensuite générer le fichier .htpasswd. Nous avons prévu dans le fichier htaccess de mettre le fichier .htpasswd dans un répertoire non accessible par un utilisateur web pour des raisons de sécurité.

```shell
root@tuto:/var/www# htpasswd -cb /var/.htpasswd pseudo motdepasse
```
Puis nous allons éditer le site par défaut de Apache2 (*/etc/apache2/sites-available/default*) et modifier la directive `AllowOverride` afin d’autoriser la modification de sa configuration par des fichiers htaccess.  
Pour ce faire, modifiez la ligne « [AllowOverride][apacheAllowOverride] » (qui est normalement sur « none ») pour la passer sur « AuthConfig »

![Configuration de l'hôte virtuel par défaut][image1]  
*Configuration de l’hôte virtuel par défaut*

Il ne nous reste plus qu’à redémarrer le serveur Apache pour que la configuration soit prise en compte.

Et voila c’est installé, vous pouvez vous connecter à votre plateforme grâce à vos identifiants. Si vous voulez en savoir plus sur le apache cliquer [ici][apacheDocumentation].

Merci Ravi Saive pour le [tutoriel source][tutorielSource]

[apacheAllowOverride]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#allowoverride "Documentation de la directive AllowOverride"
[apacheAuthType]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#authtype "Documentation directive AuthType"
[apacheDocumentation]: http://httpd.apache.org/docs/2.4/ "Documentation Apache"
[linuxdash]: https://github.com/afaqurk/linux-dash/blob/master/README.md "README de Linux Dash"
[tutorielLAMP]: https://blog.remyj.fr/linux/serveur-web-apache-php-mysql/ "Tutoriel d'installation d'un serveur web"
[tutorielSource]: http://www.tecmint.com/monitors-linux-server-performance-remotely-using-web-browser/ "Tutoriel source par Ravi Saive"

[image1]: /img/posts/monitoring-avec-linux-dash/linux-dash-1.png "Configuration de l'hôte virtuel par défaut"
