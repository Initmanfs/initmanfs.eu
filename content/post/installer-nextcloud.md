+++
author = "remy"
categories = ["linux", "web"]
date = "2016-12-01"
description = "Dans ce tutoriel, vous apprendrez à installer Nextcloud sur un serveur web. Il permet également de migrer de Owncloud vers Nextcloud."
featured = "nextcloud-logo.png"
featuredalt = "Logo Nextcloud"
featuredpath = "/img/posts/installer-nextcloud"
linktitle = ""
title = "Installer Nextcloud"
type = "post"

+++

Nextcloud est une alternative opensource aux solutions de cloud présentées par les GAFAM (Google, Apple, Facebook, Amazon et Microsoft). Outre le stockage (et partage) de fichier, il vous permettra également d’héberger votre agenda, vos contacts, votre liste de tache, …  . Nextcloud est en fait un fork de Owncloud, qui a été créé suite à un conflit d’intérêts entre Frank Karlitschek (co-fondateur et CTO d’Owncloud Inc. à l’époque) et Owncloud Inc. Une bonne partie des « grands » développeurs de Owncloud ont également changé le fusil d’épaule pour rejoindre l’équipe de Nextcloud. Actuellement, la procédure qui est décrite ci-dessous fonctionne donc pour Nextcloud et Owncloud (comme Nextcloud est encore jeune).

Afin de commencer ce tutoriel, nous allons d’abord voir comment préparer son instance Owncloud afin de la migrer vers Nextcloud. Si vous ne possédez pas encore d’instance, passez directement à la partie *[3. Installation](#3-installation-de-nextcloud)*.

Les présentations étant faites, nous pouvons donc nous préparer à installer le bébé.

## 1. Prérequis

Nextcloud est un logiciel codé en PHP. Il nécessite donc un serveur web pour le faire fonctionner. Concernant la base de données utilisé par le site, vous avec le choix entre SQLite (à n’utiliser que pour les très petites instances), MySQL/MariaDB ou PostgreSQL. Dans ce tutoriel, nous nous baserons donc sur la configuration suivante :

- Système d’exploitation : Debian 8 (Jessie)
- Serveur web : Apache 2.4 installé suivant [ce tutoriel][tutorielLAMP]
- Serveur de bases de données : MariaDB 10
- Hôte virtuel [correctement configuré][tutorielVhosts] et [avec un certificat SSL][tutoLetsEncrypt] avec le domaine *cloud.initmanfs.eu*, utilisant le dossier */var/www/cloud.initmanfs.eu* pour stocker l’instance Nextcloud.
- Utilisateur d’Apache : *www-data*

## 2. Préparation de l’instance Owncloud avant de migrer vers Nextcloud

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

## 3. Installation de Nextcloud

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
