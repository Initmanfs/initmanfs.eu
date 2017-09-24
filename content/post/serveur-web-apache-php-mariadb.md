+++
author = "remy"
categories = ["linux", "web"]
date = "2016-09-24"
description = "Tutoriel pour installer et configurer un serveur web Apache avec PHP et MariaDB sur Linux."
featured = "webserver.png"
featuredalt = "Bannière de l'article"
featuredpath = "/img/posts/serveur-web-apache-php-mariadb"
linktitle = ""
title = "Installer un serveur web Apache PHP et MariaDB"
type = "post"

+++

Dans ce tutoriel, nous allons voir comment installer un serveur web grâce à Apache et MySQL.

Ce tutoriel a été mis à jour afin de fonctionner sous Debian Jessie (Debian 8)

## Installation :

Un serveur web complet est composé de plusieurs softs :

- Un serveur de base de données, qui sera [MariaDB][mariadb] (Mais vous pouvez également utiliser MySQL, il n’y a aucune différence à la date de cet article)
- [PHP5][php] qu’Apache utilisera pour traiter le code php
- Un serveur HTTP qui servira a traiter les requêtes : [Apache2][apache2]

<!--more-->

Nous allons donc commencer à installer les paquets de ces 3 softs :
```
root@tuto ~/ apt-get install apache2 php5 php5-gd php5-curl php5-mysql mariadb-server php5-mcrypt
```
*php5-gd, php5-curl et php5-mcrypt sont des bibliothèques très utilisée en PHP, voila pourquoi je les inclue.*

Lors de l’installation du serveur [MariaDB][maraidb], le système vous demandera un mot de passe pour le super administrateur (root).

Afin d’ajouter un peu de sécurité au serveur de bases de données, un outil très utile est inclus dans le paquet du serveur : `mysql_secure_installation`. Cet outil est simplement un script interactif permettant d’effectuer les opérations de base de sécurisation de votre serveur.  

Après l’avoir lancé, il vous poseras plusieurs question :

1. Votre mot de passe root de base de données (forcément, si on veut faire une opération de maintenance sur le serveur, c’est mieux d’être en super-utilisateur)

2. La première vrai question est donc de savoir si vous voulez changer votre mot de passe root. Comme il a été défini quelques minutes plus tôt, ce n’est pas spécialement nécessaire.

3. Deuxième question : est-ce que vous voulez supprimer l’utilisateur anonyme ? Bien évidement, vous ne souhaitez pas que n’importe qui puisse se connecter à votre serveur sans avoir à entrer de login/mot de passe. Donc on va répondre oui à cette question.

4. Question suivante : désactiver l’accès root à distance ? Dans le cas d’un serveur de développement/de test disponible seulement en local, cela n’est pas nécessaire. Mais dans le cas d’un serveur de production, il est fortement conseillé de désactiver l’accès aux bases de données à distance.

5. Dernière question : suppression des bases de données de test ainsi que leurs utilisateurs ? Sauf si vous souhaitez les utiliser, vous pouvez les supprimer.

La sécurisation du serveur de bases de données étant terminé, on va pouvoir tester le fonctionnement de votre serveur apache en entrant l’ip de votre serveur dans votre navigateur ([localhost](http://localhost/ "Testez votre installation d'Apache") ou [127.0.0.1](http://127.0.0.1/ "Testez votre installation d'Apache") si vous avez installé les softs sur la machine que vous utilisez).

![Page par défaut de Apache][image1]  
*Page par défaut de Apache*

Nous allons maintenant installer [PHPMyAdmin][phpmyadmin] qui nous permettra d’avoir une interface web pour gérer le serveur de base de données :

1. Rendez-vous sur le [site de PHPMyAdmin rubrique téléchargement][phpmyadminDownload] et copiez le lien de la version « all languages » compressée en tar.gz (pour pouvoir l’avoir en francais)

2. Téléchargez l’archive :

    ```shell
    wget -O /tmp/pma.tar.gz https://files.phpmyadmin.net/phpMyAdmin/4.6.4/phpMyAdmin-4.6.4-all-languages.tar.gz
    ```

3. Il faut maintenant extraire l’archive. Nous allons l’extraire à la racine du dossier web d’Apache : */var/www* :

    ```shell
    tar -xvf /tmp/pma.tar.gz -C /var/www
    ```

    *Détail des options :  
    x : permet d’extraire l’archive  
    v : permet d’afficher les details sur l’extraction  
    f : permet de choisir le fichier à extraire*
4. L’archive contenait un dossier appelé « phpMyAdmin-4.2.8-all-languages », qui n’est pas très facile de retenir. Nous allons donc le renommer en « phpmyadmin » :
    ```shell
    mv /var/www/phpMyAdmin-4.2.8-all-languages/ /var/www/phpmyadmin
    ```

5. Vous pouvez donc essayez de vous connecter à votre base de données depuis PHPMyAdmin en entrant cette adresse dans le navigateur : <http://votre_ip/phpmyadmin/>  
    *(Connectez-vous avec les identifiants root que vous avez saisi lors de l’installation du serveur de bases de données)*

![Accueil de PHPMyAdmin][image2]  
*Accueil de PHPMyAdmin*

## Configuration :

Nous allons maintenant configurer un peu ces logiciels :

### Apache :

*La configuration d’Apache se trouve dans le fichier /etc/apache2/apache2.conf ([documentation de la configuration][apacheDoc]) et est complétée par les fichiers dans le dossier /etc/apache2/conf.d/.*

Pour commencer, je vous conseille de décommenter la ligne `addDefaultCharset` dans le fichier */etc/apache2/conf.d/charset* et de la définir (si ce n’est déjà fait) sur UTF-8. Cela évitera d’avoir des erreurs d’encodage de caractères si vous oubliez le meta charset dans votre code HTML.  
Ensuite, si votre serveur web est utilisé comme serveur de production, désactivez l’option « [serverSignature][apache2ServerSignature] » et passez l’option « [ServerToken][apache2ServerToken] » sur `prod` dans le fichier */etc/apache2/conf.d/security*  
Enfin, si vous utilisez la ré-écriture d’URL, activez le module grâce à la commande `a2enmod` :

```shell
root@tuto ~/ a2enmod rewrite
```

Vous pouvez maintenant redémarrer Apache :

```shell
root@tuto ~/ systemctl restart apache2
```

### PHP :

*La configuration de PHP5 pour Apache est dans le fichier /etc/php5/apache2/php.ini*

Tout comme pour Apache, je vous conseil d’utiliser le charset UTF-8 en décommentant la ligne « [default_charset][phpDefault_charset] »  
Encore une fois, si votre serveur est un serveur de production, désactivez l’option [expose_php][phpExpose_php].  
Rechargez les configurations de Apache :

```shell
root@tuto ~/ systemctl reload apache2
```

### PHPMyAdmin :  
Si vous n’avez pas spécifié de mot de passe de l’utilisateur MySQL lors de l’installation, PHPMyAdmin n’acceptera pas la connexion.  
Pour palier ce problème, éditez le fichier de configuration (*/var/www/phpmyadmin/config.inc.php*). Si le fichier n’existe pas, renommez le fichier */var/www/phpmyadmin/config.sample.inc.php* en config.inc.php.  
Modifiez la ligne `AllowPassword` pour l’autoriser :

```php
$cfg['Servers'][$i]['AllowNoPassword'] = true;
```

Enregistrez le fichier et vous pouvez maintenant utiliser PHPMyAdmin sans forcément avoir de mot de passe sur les utilisateurs MySQL.

Félicitation, vous avez maintenant un serveur web opérationnel.

Liens utiles :

- [Documentation sur le fichier de configuration PHP][phpDoc]
- [Documentation sur la configuration d’Apache2][apacheDoc]
- [Documentation de PHPMyAdmin][phpmyadminDoc]
- [Documentation sur la configuration de MariaDB][mariadbDoc]

[apache2]: https://httpd.apache.org/ "Site d'Apache2"
[apacheDoc]: https://httpd.apache.org/docs/2.4/fr/mod/core.html "Documentation sur la configuration d'Apache2"
[apache2ServerSignature]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#serversignature "Documentation de la directive ServerSignature"
[apache2ServerToken]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#servertokens "Documentation de la directive ServerToken"
[mariadb]: https://mariadb.com/ "Site officiel de MariaDB"
[mariadbDoc]: https://mariadb.com/kb/en/mariadb/configuring-mariadb-with-mycnf/ "documentation sur la configuration de MariaDB"
[php]: http://php.net/ "Site de PHP"
[phpDefault_charset]: http://php.net/default-charset "Documentation de l'option default_charset"
[phpDoc]: http://fr2.php.net/manual/fr/ini.core.php "Documentation sur le fichier de configuration PHP"
[phpExpose_php]: http://php.net/expose-php "Documentation de l'option expose_php"
[phpmyadmin]: http://www.phpmyadmin.net/ "Site de PHPMyAdmin"
[phpmyadminDoc]: http://docs.phpmyadmin.net/fr/latest/ "Documentation de PHPMyAdmin"
[phpmyadminDownload]: http://www.phpmyadmin.net/home_page/downloads.php "Téléchargement de PHPMyAdmin"

[image1]: /img/posts/serveur-web-apache-php-mariadb/apache2-debian-default-page-it-works.png "Page par défaut de Apache"
[image2]: /img/posts/serveur-web-apache-php-mariadb/pma-index.png "Accueil de PHPMyAdmin"
