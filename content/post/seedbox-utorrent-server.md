+++
author = "remy"
categories = ["linux"]
date = "2014-06-12"
description = "Tutoriel d'installation d'une seedbox sur Debian Wheezy grâce à µTorrent server. Seedbox fonctionnelle en moins de 30 minutes !"
featured = "seedbox1.png"
featuredalt = "Image seedbox"
featuredpath = "/img/posts/seedbox-utorrent-server/"
linktitle = ""
title = "Seedbox avec µTorrent server (Debian)"
type = "post"

+++

*Ce tutoriel est seulement fait pour installer une seedbox afin de partager des fichiers (photos, …) entre amis/famille 😉*

Après plusieurs mois d’utilisations, je me suis rendu compte que ma seedbox sous [Transmission][transmissionWebSite] ne « seedait » pas tant que ça (à peine quelques centaines de kilo-octets par seconde). Je suis donc passé sous [µTorrent server][utorrentDownload] qui m’offre de bien meilleurs résultat.Avant de rentrer plus dans les détails de l’installation, voici quelques avantages de µTorrent par rapport à Transmission :

- Recherche de peers / connexion aux peers : Le gros point noir que j’ai trouvé à Transmission est qu’avec des réglages de pare-feu trop strict, rien ne sort du serveur.
- Paramétrage : µTorrent est beaucoup plus paramétrable que Transmission.
- Interface : µTorrent nous permet de visualiser et de télécharger les fichiers directement dans le navigateur, donc pas besoin d’avoir installé un serveur FTP ou d’utiliser le SFTP pour récupérer les fichiers. On peut également voir la courbe du débit (montant et descendant).

<!--more-->

Passons maintenant à l’installation de la seedbox :

## 1. Téléchargement de µTorrent server :

Rendez-vous sur le site officiel de µTorrent pour y récupérer le lien de téléchargement (tous les exemples seront expliqués avec la version 64bits de Debian 7).Téléchargez ensuite l’archive dans le dossier de votre choix (personnellement, j’utilise le dossier /tmp pour que les fichiers soit supprimés au démarrage) :

```shell
root@tuto:/tmp# wget -O utserver.tar.gz http://download-new.utorrent.com/endpoint/utserver/os/linux-x64-debian-7-0/track/beta/
```

Vous venez donc de télécharger une archive qu’il faut extraire :

```shell
root@tuto:/tmp# tar -xvf utserver.tar.gz
```

Détail des options :

- x : permet d’extraire l’archive
- v : permet d’afficher les details sur l’extraction
- f : permet de choisir le fichier à extraire

Vous possédez maintenant un dossier nommé utorrent-server-alpha-v3_3 (au jour du tutoriel).Déplacez ensuite le dossier dans le répertoire /opt (je le renomme par la même occasion) :
```shell
root@tuto:/tmp# mv utorrent-server-alpha-v3_3/ /opt/utorrent/
```
## 2. Installation

Notre dossier étant maintenant placé au bon endroit, nous allons maintenant ajouter le programme utserver à la liste des commandes en créant un lien symbolique :

```shell
root@tuto:/tmp# ln -s /opt/utorrent/utserver /usr/bin/utserver
```

Par défaut, le script n’est pas executable. Il faut donc accorder cette permission :

```shell
root@tuto:/tmp/# chmod +x /opt/utorrent/utserver
```

Nous pouvons exécuter le serveur µTorrent pour la première fois :

```shell
root@tuto:/tmp# utserver -settingspath /opt/utorrent/
```

Si tout se passe bien, vous devriez avoir accès à l’interface web : [http://votreIp:8080/gui/](http://votreIp:8080/gui/)
Le login est admin et il n’y a pas de mot de passe.

![Interface web de µTorrent][utorrentWebGUI]  
*Interface web de µTorrent*

Vous pouvez stopper le processus avec `CTRL+C`

## 3. Lancement du script au démarrage

Pour lancer notre serveur µTorrent au démarrage, nous allons utiliser un autre utilisateur que root pour davantage de sécurité. Pour ce faire, nous utiliserons la commande useradd en définissant le répertoire personnel de l’utilisateur (/home/telechargements dans mon cas) :

```shell
root@tuto:/tmp# useradd -d /home/telechargements -m -s /bin/false utdaemon
```

Nous allons maintenant pouvoir ajouter le script pour que le serveur µTorrent se lance au démarrage de la machine en créant le fichier /etc/init.d/utserver avec le script suivant :
```bash
#!/bin/sh -e
### BEGIN INIT INFO
# Provides: utserver
# Required-Start: $local_fs $remote_fs $network
# Required-Stop: $local_fs $remote_fs $network
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Start or stop the utserver.
### END INIT INFO

CHDIR=/opt/utorrent/
NAME=utserver
OPTIONS="-LOGFILE"
DAEMON=$CHDIR/$NAME
user=utdaemon
PIDFILE=/var/run/$NAME.pid
STOP_TIMEOUT=5

[ -x $DAEMON ] || exit 1

. /lib/lsb/init-functions

start_daemon () {
pgrep -f $DAEMON >/dev/null && echo "$NAME is already running" && log_end_msg 1
rm -f $PIDFILE >/dev/null
start-stop-daemon --start --quiet --make-pidfile --pidfile $PIDFILE --chuid $user --chdir $CHDIR --background --exec $DAEMON -- $OPTIONS
}

case "$1" in
start)
log_daemon_msg "Starting $NAME daemon" "$NAME"
start_daemon
log_end_msg 0
;;
stop).fr
log_daemon_msg "Stopping $NAME daemon" "$NAME"
start-stop-daemon --stop --quiet --exec $DAEMON --retry $STOP_TIMEOUT || log_end_msg 1
pgrep -U $user $NAME >/dev/null || rm -f $PIDFILE >/dev/null
log_end_msg 0
;;
restart)
log_daemon_msg "Restarting $NAME daemon" "$NAME"
start-stop-daemon --stop --quiet --pidfile $PIDFILE --exec $DAEMON --retry $STOP_TIMEOUT || log_end_msg 1
start_daemon
log_end_msg 0
;;
*)
echo "Usage: /etc/init.d/$NAME {start|stop|restart}"
exit 2
;;
esac

exit 0
```

Après avoir créé le fichier, il faut rendre le script exécutable puis l’ajouter aux routines de démarrage et d’arrêt grâce à la commande *update-rc.d* :
```shell
root@tuto:/tmp# chmod +x /etc/init.d/utserver
root@tuto:/tmp# update-rc.d utserver defaults
```
Et voilà, normalement en redémarrant votre machine, le serveur µTorrent devrait démarrer automatiquement.

[transmissionWebSite]: https://www.transmissionbt.com/
[utorrentDownload]: http://www.utorrent.com/downloads/linux

[utorrentWebGUI]: /img/seedbox-utorrent-server/utorrent-gui.png
