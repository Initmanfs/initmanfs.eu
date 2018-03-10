+++
author = "remy"
categories = ["linux", "web"]
date = "2016-12-01"
description = "In this recipe, you will learn to install Nextcloud on a web-server."
featured = "baniereEN.svg"
featuredalt = "Logo Nextcloud"
featuredpath = "/img/posts/installer-nextcloud"
linktitle = ""
title = "Install Nextcloud"
type = "post"
slug = "install-nextcloud"
+++

Nextcloud is an opensource alternative to GAFAM's cloud solution. Besides storing and sharing files, it enable you to store Calendars, contacts, task list and many more.
Nextcloud is a fork from Owncloud, it was born because of a conflict of interest between Frank Karlitschek, cofounder and CTO of Owncloud Inc. at the time and Owncloud Inc. Many of the core developer of Owncloud followed Frank into Nextcloud. As of now the procedure to install Nextcloud is the same than the one to install Owncloud, since the fork is recent.

In order to start neatly, we are going through the prerequisite of migrating your Owncloud instance to a Nextcloud one. If you do not have any instance yet, jump too *[3. Installation](# 3. Installation of Nextcloud)*.

<!--more-->

Now, lets dive right in.

## 1. Prerequisite

Nextcloud is a software coded in PHP. It thus needs a web-server. To store data, you can choose between SQLite (only for small instances), MySQL/MariaDB or PostgreSQL. It this tutorial we will use this configuration :

- Operating System : Debian 8 (Jessie)
- Web Server : Apache 2.4 installed with [this tutorial][tutorielLAMP]
- DataBase Server : MariaDB 10
- Virtual Host [configured properly][tutorielVhosts] and [with a SSL certificat][tutoLetsEncrypt] for the domain *cloud.initmanfs.eu*, using the folder */var/www/cloud.initmanfs.eu* to store the Nextcloud instance.
- Apache user : *www-data*

## 2. Préparation of Owncloud before migration to Nextcloud

La préparation de notre ancienne instance avant la migration est assez simple : elle se résume au nettoyage du dossier Owncloud puis, à l’installation de Nextcloud, en se basant sur la configuration de Owncloud. La fin de la procédure étant commune à une installation en partant de rien, nous ne verrons dans cette partie le nettoyage du dossier Owncloud afin de préparer la place à Nextcloud.

Nous partirons donc du postula que vous avez actuellement une instance Owncloud installée dans votre racine de l’hôte virtuel Apache, utilisant le dossier data à sa racine pour stocker les fichiers déjà présent sur votre instance.

Votre dossier */var/www/cloud.initmanfs.eu/* devrait donc ressembler à cela :

- apps : Toutes vos applications
- config (contenant config.php et config.sample.php)
- core
- data : Tous les fichiers de vos utilisateurs et les logs
- l10n
- lib
- ocs
- ocs-provider
- resources
- settings
- updater
- plusieurs fichiers

Ce qui va nous intéresse de garder, c’est le dossier data (qui contient doutes les données de vos utilisateurs) et le fichier *config/config.php*.  
<span style="color: #ff6600;"> Attention, avant de faire quoi que ce soit, pensez à sauvegarder toutes vos données (y compris la base de données).</span>  
Garder également la liste des applications que vous avez installée, cela nous servira par la suite.

Avant de commencer, nous allons passer l’instance en mode de maintenance, afin d’éviter toute interaction d’un quelconque utilisateur :

```shell
remy@tuto:/var/www/cloud.initmanfs.eu/$ sudo -u www-data ./occ maintenance:mode --on
```

Comme les fichiers de Owncloud vont être supprimer, l’utilisateur va se retrouver sur une erreur 403, 404 ou la liste de vos fichiers, suivant la configuration de votre serveur web. Pour prévenir à ce fait, nous allons donc créer un petit fichier HTML :

```shell
remy@tuto:/var/www/cloud.initmanfs.eu/$ echo "Ce service est actuellement en maintenance. Merci de revenir dans quelques instants." > index.html
```

Nous pouvons donc maintenant purger le dossier en toute tranquillité. Pour ce faire, petite commandes magique (vous devez utiliser bash comme shell):

```shell
remy@tuto:/var/www/cloud.initmanfs.eu/$ shopt -s extglob
remy@tuto:/var/www/cloud.initmanfs.eu/$ sudo rm -fr -- !(config|data|index.html)
```

Vous pouvez vérifier, le dossier ne contient plus que notre fichier HTMl et les dossiers config et data. Nous pouvons donc maintenant passer à l’installation de Nextcloud.

## 3. Installation of Nextcloud

L’installation de Nextcloud va se dérouler en 4 étapes. La première consistera à télécharger et installer Nextcloud, la seconde à créer la base de données et la dernière à suivre l’assistant pour configurer l’instance.

Commençons donc par l’installation de Nextcloud. Rendez-vous sur le [site de Nextcloud][nextcloud] dans la section téléchargement pour récupérer le lien de l’archive (cliquez sur « Details and download options » pour avoir le lien en format tar.bz2). Une fois le lien récupérer, nous allons utiliser la commande suivante pour télécharger et extraire l’archive dans le dossier */tmp* :

```shell
remy@tuto:/tmp/$ wget https://download.nextcloud.com/server/releases/nextcloud-10.0.1.tar.bz2 -O - | tar -xj
```
L’archive étant décompressée, nous pouvons maintenant déplacer son contenu dans le bon dossier :

```shell
remy@tuto:/var/www/cloud.initmanfs.eu/$ sudo mv /tmp/nextcloud/* /tmp/nextcloud/.* /var/www/cloud.clog.remyj.fr/
```

Comme ce n’est pas nous, mais l’utilisateur *www-data* qui va utiliser les fichiers, nous allons le nommer propriétaire. Par la même occasion, nous allons autoriser l’exécution de la console (occ) et du script lancé par le cron (*crop.php*) :

```shell
remy@tuto:/var/www/cloud.initmanfs.eu/$ chown -R www-data:www-data ./
remy@tuto:/var/www/cloud.initmanfs.eu/$ sudo chmod u+x cron.php occ
```

Pour les personnes suivant la procédure de migration, passez directement à la partie suivante : [*4. Fin de la migration vers Nextcloud*](#4-fin-de-la-migration-vers-nextcloud)

Les fichiers étant prêts, nous allons maintenant créer la base de données. Pour ce faire, exécuter la commande ci-dessous. Elle permet de créer un utilisateur (pensez à changer le mot de passe) et de lui accorder tous les droits sur une base fraichement créée :

```shell
remy@tuto:/var/www/cloud.initmanfs.eu/$ echo "CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'yourpassword';GRANT USAGE ON *.* TO 'nextcloud'@'localhost' REQUIRE NONE WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0;CREATE DATABASE IF NOT EXISTS \`nextcloud\`;GRANT ALL PRIVILEGES ON \`nextcloud\`.* TO 'nextcloud'@'localhost';" | mysql -u root -p"mot_de_passe_utilisateur_root_mysql"
```

Tout est maintenant prêt. Il ne nous reste qu’à nous rendre sur notre url pour avoir l’assistant de configuration :

![Assistant d'installation Nextcloud][image1]  
*Assistant d’installation Nextcloud*

Comme vous pouvez le voir, rien de bien méchant.  
A ce stade, vous avez terminé l’installation. Mais il reste seulement un petit point que j’aime bien mettre en place : l’exécution des tâches planifiées via le cron. Cela permet d’éviter que les pages mettent trop de temps à se charger. Pour cela, rendez-vous dans l’administration, et sélectionnez Cron dans la partie Cron. Ajoutez ensuite la ligne suivante au crontab de l’utilisateur *www-data* (*crontab -eu www-data*) :

```
*/15 * * * * php -f /var/www/cloud.initmanfs.eu/cron.php
```

Et voila, votre instance Nextcloud est maintenant opérationnelle.

## 4. Fin de la migration vers Nextcloud

Maintenant que Nextcloud est correctement installé, il ne nous reste plus qu’à lancer la mise à jour depuis la console :

```shell
remy@tuto:/var/www/cloud.initmanfs.eu/$ sudo -u www-data php occ upgrade
remy@tuto:/var/www/cloud.initmanfs.eu/$ sudo -u www-data php occ maintenance:mode --off
```

Si la commande d’upgrade échoue en vous disant que vous ne pouvez pas downgrader, modifiez la version dans le fichier de configuration afin qu’elle soit inférieure à la version de Nextcloud

La migration est maintenant terminée. Pensez à réinstaller vos applications.

[nextcloud]: https://nextcloud.com/ "Site officiel de Nextcloud"
[tutorielLAMP]: https://blog.remyj.fr/linux/serveur-web-apache-php-mysql/ "Tutoriel d'installation d'un serveur web"
[tutoLetsEncrypt]: https://blog.remyj.fr/linux/activer-https-apache-lets-encrypt/ "Tutoriel de sécurisation HTTPS avec Let's Encrypt"
[tutorielVhosts]: https://blog.remyj.fr/linux/ajouter-des-hotes-virtuels-sur-un-serveur-apache/ "Tutoriel de configuration des hôtes virtuels"

[image1]: /img/posts/installer-nextcloud/nextcloud-installation.png "Assistant d'installation Nextcloud"
