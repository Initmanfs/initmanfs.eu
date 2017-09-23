+++
author = "remy"
categories = ["linux", "web"]
date = "2016-10-01"
description = "Apprenez à faire un hôte virtuel (virtualhost) sur Apache2"
featured = "how_to_create_a_virtual_host.jpg"
featuredalt = "Illustration des virutalhosts"
featuredpath = "/img/posts/ajouter-des-hotes-virtuels-sur-un-serveur-apache"
linktitle = ""
title = "Ajouter des hôtes virtuels sur un serveur Apache2"
type = "post"

+++

# Introduction

Un hôte virtuel ou virtual host (abrégé vhost) en anglais est un système permettant au serveur web (Apache dans notre cas, mais présent sur la majorité des serveurs webs) d’héberger plusieurs sites, avec leurs paramètres respectifs, sur un seul et même serveur. Sans ce système, nous serions obligé d’utiliser un serveur différent par site. Vous avouerez que ce n’est pas spécialement rentable d’avoir donc un serveur (qu’il soit physique ou virtuel) par site : devoir avoir un noyau de système qui tourne juste pour un petit site, c’est pas top.


## Fonctionnement d’un hôte virtuel

Sur le papier, comment fonctionne un hôte virtuel ?

Tous les serveurs web utilisent le même système pour définir sur quel hôte se baser lorsqu’il reçoit une requête : le nom de domaine complètement qualifié (FQDN). Le FQDN est le nom de domaine complet d’un site (pour l'exemple, nous utiliserons tutoriel.initmanfs.eu).

![Schéma FQDN][image1]  
*Explication du FQDN*

Tous les navigateurs récents envoient le FQDN demandé dans les requêtes au serveur web. Ainsi, le serveur sait vers quel hôte virtuel se tourner.


# Avec Apache, comment ça se passe ?

Apache permet aux administrateurs de faire deux type d’hôtes virtuels :

- Les hôtes virtuels basés sur l’IP : si le serveur a plusieurs IPs de configurées, Apache peut gérer des hôtes différents suivant l’IP par laquelle on passe pour transmettre la requête.
- Les hôtes virtuels basés sur le nom de domaine : comme expliqué précédement, Apache se base sur le FQDN pour gérer les requêtes. C’est le type d’hôtes virtuels le plus utilisé.


## Les fichiers de configuration

Afin de mener à bien ce tutoriel, vous devez avoir installé un serveur Apache. Si vous ne savez pas comment vous y prendre, il y a justement un [tutoriel prévu pour ça.][tutorielLAMP].

Dans le paquet Debian (et Ubuntu) de Apache, les fichiers de configuration de Apache (*/etc/apache2*)  pour les hôtes virtuels sont préparés comme ceci :

- *sites-available* : On y trouve dedans toutes les configurations d’hôtes virtuels. C’est donc dans ce dossier que vous allons travailler.
- *sites-enable* : Ce dossier est rempli avec des liens symboliques vers les hôtes virtuels disponibles qui sont activés. C’est le dossier dans lequel Apache récupère les hôtes virtuels.

Si vous explorez ces deux dossiers, vous pouvez voir que vous avez deux hôtes qui sont présents (*000-default.conf* et *default-ssl.conf*), mais que seul *000-default.conf* est activé. *000-default.conf* correspond à l’hôte qui est utilisé par défaut par Apache. *default-ssl.conf* correspond à une configuration de base pour le HTTPS. Mais ce n’est pas le sujet de ce tutoriel.


## Compréhension d’une configuration d’hôte virtuel

Afin de créer notre hôte virtuel, nous allons nous baser sur la configuration de l’hôte par défaut (que j’ai un peu épuré) :

```apache
<VirtualHost *:80>
    #ServerName www.example.com

    ServerAdmin webmaster@example.com
    DocumentRoot /var/www/html

    #LogLevel info ssl:warn

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Quelques petites explications s’impose :

### La balise [`<VirtualHost>`][apache2VirtualHost]

Comme dans le HTML ou le XML, les configurations d’Apache utilisent des blocs afin de regrouper certaines directives. Comme on le voit ici, la balise d’hôte virtuel est ouverte en début de configuration pour être fermée en fin de configuration. Donc toutes les directives entre ces deux balises ne seront associées qu’à cet hôte virtuel.

De plus, on peut voir que dans la balise d’ouverture du bloc, qu’il y a un paramètre : `*:80`. Cela signifie que l’hôte virtuel est valide pour toutes les requêtes arrivant sur le port 80 du serveur, quelque soit sont IP. C’est ce paramètre que l’on modifie pour définir un hôte virtuel basé sur une IP. Par exemple, on aurait pu avoir « 12.34.56.78:80 ». Cela aurait signifié que seul les requêtes arrivant sur l’IP 12.34.56.78 et sur le port 80 aurait été pointées vers cet hôte virtuel.

### ServerName

La première directive que l’on découvre est le "[ServerName][apache2ServerName]" . Comme la configuration provient de l’hôte par défaut, il ne faut pas le limiter à un seul FQDN, donc la directive est commentée. Dans notre cas, il nous faut justement la décommenter et indiquer le FQDN que nous souhaitons utiliser pour cet hôte. Vous pouvez également utiliser la directive "[ServerAlias][apache2ServerAlias]" pour indiquer plusieurs autres FQDN qu’il faut utiliser pour cet hôte.

### ServerAdmin

Nous avons ensuite la directive "[ServerAdmin][apache2ServerAdmin]" . Elle permet d’indiquer l’adresse email de l’administrateur du serveur dans les pages d’erreur.

### DocumentRoot

La directive "[DocumentRoot][apache2DocumentRoot]" est l’une des plus importantes dans la configuration de l’hôte : elle permet de définir vers quel dossier pointe l’hôte virtuel. Sans ça, le dossier par défaut (/var/www/html) sera utilisé.  
Personnellement (et comme beaucoup de gens), je place tous les dossiers de mes sites dans un dossier commun (/var/www par exemple), puis je crée un sous-dossier par hôte, portant le nom du FQDN de l’hôte. Cela permet de savoir d’un coup d’œil quel dossier appartient à quel hôte.

### LogLevel

La directive suivante est commentée car elle n’a pas beaucoup d’utilité. Le "[LogLevel][apache2LogLevel]" permet simplement de définir qu’est-ce qui va être présent dans les logs d’erreurs. Laisser la valeur par défaut convient parfaitement.

### ErrorLog et CustomLog

Et enfin, les deux dernières directives concernent l’emplacement des fichiers de log. "[ErrorLog][apache2ErrorLog]" permet de définir l’emplacement du fichier de journalisation d’erreur quand à "[CustomLog][apache2CustomLog]" , elle permet de définir le fichier des journaux des requêtes envoyées au serveur. Tout comme pour les dossier, j’ai bien avoir un fichier de log différent par hôte. Cela permet d’avoir des fichiers sans surplus.

Si vous souhaitez plus de détails sur toutes les directives disponibles, un seul lien : [la documentation de Apache][apache2Doc]

## Notre premier hôte virtuel

Maintenant que vous avez compris l’utilité de chaque directive, vous pouvez donc adapter la configuration à votre guise. Pour ma part, ça donne cette configuration :

```apache
<VirtualHost *:80>
    ServerName tutoriel.initmanfs.eu

    ServerAdmin administrateur@initmanfs.eu
    DocumentRoot /var/www/tutoriel.initmanfs.eu

    #LogLevel info ssl:warn

    ErrorLog ${APACHE_LOG_DIR}/tutoriel.initmanfs.eu-error.log
    CustomLog ${APACHE_LOG_DIR}/tutoriel.initmanfs.eu-access.log combined
</VirtualHost>
```

Nous allons donc pouvoir créer notre fichier de configuration dans le dossier sites-available. A savoir que depuis Debian 8, la configuration d’Apache est faite de sorte à se que les fichiers de configuration doivent se terminer par l’extension *.conf*. De plus, comme pour les logs et les dossiers, j’ai comme convention de nommer mes fichiers de configuration d’hôtes virtuels avec le FQDN relatif à l’hôte.

Dans mon cas, je vais donc placer ma configuration dans le fichier */etc/apache2.sites-available/tutoriel.initmanfs.eu.conf*

Une fois la configuration en place, il ne nous reste plus qu’à l’activer. Pour cela, un script est fourni dans le paquet Debian (et Ubuntu), `a2ensite` :

```shell
remy@tutopia:~$ sudo a2ensite tutoriel.initmanfs.eu
```

Comme l’indique le script, il faut maintenant recharger la configuration d’Apache :

```shell
remy@tutopia:~$ sudo systemctl reload apache2.service
```

La configuration d’Apache est maintenant terminée.

# Configuration du FQDN

Mais ce n’est pas la fin de notre périple (promis, c’est bientôt fini). Il nous reste encore à faire en sorte que notre FQDN pointe vers notre serveur. Si vous êtes déjà familier avec la gestion de zone DNS, parfait, vous devriez savoir faire. Sinon, lisez bien ce qui suit.

Nous avons voir les deux cas de figures principaux : lorsque l’hôte virtuel est pour un usage local (test ou développement) et lorsqu’il est pour un usage de production (accessible par tout le monde).


## Usage local

C’est le cas le plus simple : le site n’a pas besoin d’être accessible depuis internet, il n’y a que vous qui y accédez. La procédure est très simple :

### Sur Windows

1. Lancer le bloc-note (ou votre éditeur de texte favoris) en tant qu’administrateur

2. Ouvrez le fichier *C:\Windows\System32\Drivers\etc\hosts*

3. Ajouter l’IP de votre serveur suivi du FQDN à la fin du fichier, les deux séparés par un espace :

    ```
    12.34.56.78 tutoriel.initmanfs.eu
    ```

4. Sauvegardez le fichier

### Sur Linux et Mac

Éditez le fichier */etc/hosts* en tant que root et ajouter l’IP de votre serveur suivi du FQDN à la fin du fichier, les deux séparés par un espace :  

```
12.34.56.78 tutoriel.initmanfs.eu
```

## Usage de production

Pour un usage de production, il ne faut pas que chaque utilisateur voulant accéder à notre hôte virtuel ai besoin de modifier son fichier hosts. Sinon, vous n’aurez aucun visiteur.

Il vous faudra donc utiliser un nom de domaine (enregistré auprès d’un registrar comme OVH, Gandi ou encore GoDaddy).

Lorsque votre domaine est enregistré et que vous pouvez le configurer, il vous faut ajouter un enregistrement de type A dans votre zone DNS pointant vers l’IP de votre serveur.

Une fois le domaine accessible par navigateur, on voit bien que l’hôte virtuel est fonctionnel.

[apache2CustomLog]: https://httpd.apache.org/docs/current/mod/mod_log_config.html#customlog "Documentation de la directive CustomLog"
[apache2Doc]: https://httpd.apache.org/docs/2.4/fr/mod/core.html "Documentation de Apache2"
[apache2DocumentRoot]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#documentroot "Documentation de la directive DocumentRoot"
[apache2ErrorLog]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#errorlog "Documentation de la directive ErrorLog"
[apache2LogLevel]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#loglevel "Documentation de la directive LogLevel"
[apache2ServerAdmin]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#serveradmin "Documentation de la directive ServerAdmin"
[apache2ServerAlias]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#serveralias "Documentation de la directive ServerAlias"
[apache2ServerName]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#servername "Documentation de la directive ServerName"
[apache2VirtualHost]: https://httpd.apache.org/docs/2.4/fr/mod/core.html#virtualhost "Documentation de la directive VirtualHost"
[tutorielLAMP]: https://blog.remyj.fr/linux/serveur-web-apache-php-mysql/ "Tutoriel d'installation d'un serveur web"

[image1]: /img/posts/ajouter-des-hotes-virtuels-sur-un-serveur-apache/FQDN.svg "Schéma FQDN"
