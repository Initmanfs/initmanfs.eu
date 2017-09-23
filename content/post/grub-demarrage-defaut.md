+++
author = "arthur"
categories = ["linux", "windows"]
date = "2016-08-05"
description = "Tutoriel montrant comment passer GRUB en démarrage par défaut à la place de Windows et comment lui ajouter une image de fond."
featured = "grub.png"
featuredalt = "Image de Grub au démarrage"
featuredpath = "/img/posts/grub-demarrage-defaut"
linktitle = ""
title = "Utiliser Grub en démarrage par défaut"
type = "post"

+++

Faire apparaître son GRUB quand vous êtes en UEFI n’est pas très compliqué cependant il faut appuyer sur F9 (pour moi) et si on le fait pas assez vite Windows démarre et là c’est long. Quitte à voir quelque chose régulièrement autant qu’il soit joli, ça ne coûte pas plus cher. Ce tutoriel sera en deux parties. La première (purement pratique) : faire que le démarrage passe par le GRUB et la deuxième, plus succincte, sera sur comment le rendre joli.

## Forcer le démarrage sur le GRUB

### Pour ceux qui ont un BIOS personnalisable

Pour ceux qui un BIOS compétent rien de plus simple. Il suffit d’aller dans le bios mettre le GRUB en premier et le tour est joué.

### Pour ceux qui n’ont pas d’option

Le programme de démarrage cherche le fichier avec l’extension EFI dans le dossier Microsoft. S’il le trouve, il démarre directement sur Microsoft Windows X et s’il ne trouver pas il se redirige sur les autres fichiers, en tout cas c’est comme cela que mon système de démarrage fonctionne (PC HP).

Utilisateur Linux avéré, j’avais trouvé une solution : désinstaller Windows. Cependant, ça ne coûte qu’un peu de mémoire de le garder, et il y peux être des jeux ou des programmes qui pourraient être utile de garder et qui fonction sous Windows, de plus il est important de garder une ouverture d’esprit envers le blagueur Microsoft.  
Je vais vous donner ma méthode pour résoudre ce problème.

J’ai tout d’abord commencé par empêcher mon programme de démarrage de trouver le fichier EFI de Microsoft, pour éviter qu’il ne démarre instantanément sans me demander la permission. Pour cela rien de plus simple, il suffit de le déplacer :

```shell
mv /boot/efi/EFI/Microsoft/ /boot/efi/EFI/Micro
```

Maintenant, il ne démarre plus sous Windows de manière automatique. Cependant comme dit plus haut nous voulons garder cette possibilité. Si nous voulons démarrer sur Windows pour être sûr de n’avoir rien cassé, vous pouvez le faire via l’option dans le menu de boot, « démarrage depuis un fichier EFI ».

Trouvons l’UUID de la partition de démarrage avec la commande: *blkid*

![Liste des UUID des disques][image1]  
*Liste des UUID des disques*

Disons à GRUB où est le fichier EFI de Microsoft. Éditons le fichier : */etc/grub.d/40_custom* .

```
menuentry "Windows 10" {
insmod part_gpt
insmod fat
insmod search_fs_uuid
insmod chain
search --fs-uuid --no-floppy --set=root VOTREUUID
chainloader /EFI/Micro/Boot/bootmgfw.efi
}
```
N’oublier pas de remplacer le chemin si vous avez mis le fichier EFI ailleurs et évidement le UUID de la partition de démarrage.  
Il nous reste plus qu’à mettre à jour GRUB avec la commande : `update-grub`.

Voila maintenant l’ordinateur démarre sur le GRUB plus besoin spam-click F9 au démarrage.

## Un joli GRUB

Nous entrons dans la deuxième partie de ce tutoriels où nous allons embellir notre GRUB.

Vous allez avoir besoin d’une joli image, une moche fonctionnera mais n’auras pas l’effet escompter, et de modifier deux ligne dans un fichier.  
Je suppose que vous avez déjà votre image. Le mieux serait de la placer avec les autre fonds d’écran de votre distribution, dans */usr/share/backgrounds* ou */usr/share/distribution/wallpaper…*. Cela évitera que vous la supprimiez par erreur et c’est également plus propre au point de vue de l’organisation des fichiers. Voici la mienne: <https://alpha.wallhaven.cc/wallpaper/274887>

Nous allons donc éditer le fichier */etc/default/grub* et nous allons insérer et/ou dé-commenter ces lignes là:
```
GRUB_GFXMODE=1980x1080
GRUB_BACKGROUND="/usr/share/lubuntu/wallpapers/favorites/wallhaven-274887.jpg"
```
N’oubliez pas de changer l’adresse de la photo et la résolution par celle que vous voulez.  
Mettons à jour notre bootloader préféré :

```shell
update-grub
```
Et voila votre GRUB est fonctionnel et joli.  
Mission accomplie !


[image1]: /content/images/2017/09/grub-1.png "Liste des UUID des disques"
