+++
author = "arthur"
categories = ["linux"]
date = "2017-09-01"
description = "Installer le client TeamSpeak, résolvez l'erreur de libquazip.so et peaufinez l'installation en rendant le logiciel disponible depuis le menu"
draft = true
featured = ""
featuredalt = ""
featuredpath = "/img/posts/installer-client-teamspeak-3-linux"
linktitle = ""
title = "Installer le client TeamSpeak 3 sous Linux"
type = "post"

+++


Bonjour, aujourd’hui on va installer Teamspeak 3 version client. La version dont on a besoin pour parler pendant des heures sur un serveur !

Téléchargement du client :

Pour télécharger Teamspeak je vous invite à aller [https://www.teamspeak.com/downloads#][teamspeakDownloads] et à télécharger la version correspondante a votre architecture.

Puis rendez-vous avec le terminal dans le dossier où se trouve votre téléchargement. Donnez les droits d’exécution au .run que vous avez téléchargé.

```shell
chmod +x TeamSpeak3-Client-linux_amd64-3.0.18.2.run
```

Une fois les droits d’exécution attribués, on peut le lancer.

```shell
./TeamSpeak3-Client-linux_amd64-3.0.18.2.run
```

Lisez les CGUs en appuyant sur entré, puis sur q pour quitter. Acceptez-les en appuyant sur y. Voila un dossier TeamSpeak est apparu qui contient le logiciel (TeamSpeak3-Client-linux_amd64 dans mon cas). Maintenant est venu le temps de le déplacer là où bon vous semble. Vous pouvez le laisser là où vous voulez. Pour ma part je vais le renommer et le déplacer dans /opt.

```shell
mv TeamSpeak3-Client-linux_amd64/ /opt/TeamSpeak3
```

Afin que Teamspeak soit exécutable par tous les utilisateurs, on va modifier les permissions du dossier.

```shell
chmod -R a+r /opt/TeamSpeak3
```

puis on lance le script d’initialisation :

```shell
/opt/TeamSpeak3 /ts3client_runscript.sh
```

On accepte les conditions et nous voila sur Teamspeak.

Puis nous allons faire un launcher pour la commande qui exécutera le launcher.

```shell
nano TeamSpeak.desktop
```

Puis copier-coller et enregistrer. notez que j’ai le pack d’icône numix d’installé ainsi donc mon icône diffère peut être de la votre.
```ini
[Desktop Entry]
Name = TeamSpeak Client
Comment = Chat with your friends for hours.
Keywords = Chat;Internet;
Exec = /opt/TeamSpeak3/ts3client_runscript.sh
Categories = Network;Chat;
Terminal = false
Type = Application
StartupNotify =true
Icon = /usr/share/icons/Numix-Circle-Light/48x48/apps/ts3client_linux_amd64.svg
```

```shell
 chmod +x TeamSpeak.desktop
```

Si vous double-cliquez sur TeamSpeak, via l’explorateur, le logiciel se lance sans avoir à taper une commande. Pratique. Cependant afin de le trouver dans votre menu nous allons le glisser dans un dossier magique.

```shell
sudo cp TeamSpeak.desktop /usr/share/applications/
```
Allez voir dans votre menu et le voila qui apparaît et qui se lance, n’est-ce pas magique ?

[teamspeakDownloads]: https://www.teamspeak.com/downloads "Page de téléchargement de Teamspeak3"
