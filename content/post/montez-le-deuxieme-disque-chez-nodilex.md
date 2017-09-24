+++
author = "remy"
categories = ["linux", "nodilex"]
date = "2015-04-21"
description = ""
featured = "nodilex-logo.png"
featuredalt = "Logo Nodilex"
featuredpath = "/img/posts/montez-le-deuxieme-disque-chez-nodilex"
linktitle = ""
title = "Montez le deuxième disque chez Nodilex"
type = "post"

+++

Dans ce tutoriels nous allons monter le deuxième fourni avec la location d’un VPS chez l’hébergeur Nodilex.

Pour commencer nous allons créé la partitions :
`fdisk /dev/sdb`

Continuons par lister les partitions présente sur le vps:
`fdisk -l`

<!--more-->

![Définition du deuxième disque][image1]  
*Définition du deuxième disque*

Nous voyons ainsi, que le disque qui nous intéresse est /dev/xvdb car il correspond à la quantité de mémoire que je suis supposé avoir avec mon offre (200 Go).

Nous allons créé la partitions :

```shell
fdisk /dev/sdb
```

Ceci devrait ouvrir un menu interactif pour créé une nouvelle partition appuyez sur `n`. Puis pour dire que c’est une partition primaire appuyer sur `p`.  
Nous allons donc formater la partition. Nous allons la formater au format ext4. Nous utiliserons pour ceci la commande `mkfs.ext4` avec l’argument `b`, qui permet définir la taille des  secteurs. La taille approprié des secteurs est de 4096 Octets.

```shell
mkfs.ext4 -b 4096 /dev/xvdb1
```

![Formatage de la partition créée sur le disque][image2]  
*Formatage de la partition créée sur le disque*

Puis nous allons éditer le fichier */etc/fstab* . Ce dernier permet de monter et de peupler le système de fichier racine au moment du démarrage du serveur.

Afin de le compléter correctement il nous faut respecter la syntaxe du fichier :

```
<système de fichier> <point de montage> <type de système de fichier> <options de montage> <dump> <pass>
```
Dans notre cas:

- le système de fichier: /dev/xvdb1
- le point de montage : /home
- le type système de fichier: ext4
- les options: errors=remount-ro ( si des erreurs surviennent lors du montage du disque, il réessaye et le monte en lecture seule.)
- le dump : 0 ( ceci permet d’ignorer le système de sauvegarde, l’autre entrée possible est 1.)
- le pass: 0 ( ceci permet que le disque ne soit pas vérifié par l’utilitaire fsck, les autres entrées possibles sont 1et 2 qui définissent l’ordre de priorité des vérifications.)

[Plus d’infos sur le fichier fstab][infofstab]

Nous allons donc ajouter à la fin de ce fichier:

```
/dev/xvdb1 /home ext4 errors=remount-ro 0 0
```

Afin d’éviter toute perte, nous allons copier les fichiers déjà présent dans le dossier dans lequel nous allons monter la nouvelle partition vers un autre dossier.

```shell
cp -rp /home/* /home_old/
```

Nous pouvons maintenant monter le disque sur /home :

```shell
mount /dev/xvdb1/ /home
```

Il ne vous reste plus qu’à recopier le contenu de l’ancien dossier home dans notre nouveau :
```shell
cp -rp /home_old/* /home/
```

Félicitations vous disposez maintenant de tout l’espace disque disponible sur votre vps !

[infoFstab]: https://wiki.archlinux.org/index.php/fstab "Wiki Archlinux sur fstab"

[image1]: /img/posts/montez-le-deuxieme-disque-chez-nodilex/disque-nodilex-1.png "Définition du deuxième disque"
[image2]: /content/images/2017/09/disque-nodilex-2.png "Formatage de la partition créée sur le disque"
