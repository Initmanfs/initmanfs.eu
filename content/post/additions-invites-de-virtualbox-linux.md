+++
author = "arthur"
categories = ["linux", "virtualisation"]
date = "2015-08-20"
description = "Ce tutoriel vous montrera comment installer les additions invités de Virtualbox sous Linux."
featured = "virtualbox-logo.png"
featuredalt = "Logo de Virtualbox"
featuredpath = "/img/posts/additions-invites-de-virtualbox-linux"
linktitle = ""
title = "Les additions invités de VirtualBox sous Linux"
type = "post"

+++

Lorsque vous utilisez VirtualBox, beaucoup de fonctionnalités sur le système virtualisé dépendent de l’installation des additions invité de VirtualBox. Ainsi dans ce tutoriel on va apprendre à installer ces fameuses additions !

Ce tutoriel commence donc une fois votre machine virtuel installé. Nous allons monter les disques dans un dossier que nous créerons à cette effet, copier ce qui nous intéresse et lancer le script.  
Créons donc le dossier :

```shell
mkdir /mnt/vb
```

Dans ce tutoriel, nous utiliserons le dossier */mnt/vb* comme point de montage mais vous pouvez l’appelez comme vous voulez.

Maintenant montons le disque. Allez dans l’onglet périphérique de la fenêtre VirtualBox et tout en bas cliquez sur insérer l’image CD des additions invité, ou sur la touche `Host` + `D` , si vous n’avez pas changer les réglages c’est la touche Contrôle de droite. Lors de l’écriture de ce tutoriel, la version des additions invités était la 4.3.26. Modifiez donc la version dans la commande suivante par la votre.

```shell
mount /dev/disk/by-label/VBOXADDITIONS_4.3.26 /mnt/vb
```

Puis on copie le contenue de vb dans */tmp*

```shell
cp -rf /mnt/vb /tmp
```

et on exécute `autorun.sh`

```shell
/tmp/vb/autorun.sh
```

Et Voila ! Ceci n’est peut être pas la méthode la plus simple mais c’est celle que j’utilise. N’oubliez pas de redémarrer pour activer toutes ces petites fonctionnalités.
