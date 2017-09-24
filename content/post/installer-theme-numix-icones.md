+++
author = "arthur"
categories = ["linux"]
date = "2016-10-15"
description = "Tutoriel montrant comment installer le thème Numix sous Linux"
featured = "tuto-numix-banniere.png"
featuredalt = "Bannière de l'article"
featuredpath = "/img/posts/installer-theme-numix-icones"
linktitle = ""
title = "Installer le thème Numix et ses icônes"
type = "post"

+++

Les thème sous Linux sont nombreux mais les thèmes de la qualité et de la finesse de Numix sont rare. Ceci peut faire toute la différence entre un environnement de bureau attrayant et un autre qui ne l’est pas ! Aujourd’hui je vais vous montrer comment l’installer.  
Comparons avec le thème et les icônes par défaut de lxde:

![tuto-numix-2][image1]  
![tuto-numix][image2]

### Numix sous Ubuntu :

```shell
sudo add-apt-repository ppa:numix/ppa
sudo apt update
sudo apt-get install numix-gtk-theme numix-icon-theme numix-icon-theme-circle
```

### Numix sous Fedora :

```shell
sudo dnf copr enable numix/numix
sudo dnf install numix-gtk-theme numix-icon-theme numix-icon-theme-circle
```

Une fois ceci effectué, vous n’avez plus qu’à aller dans votre gestionnaire de thèmes et sélectionner Numix.

[image1]: /img/posts/installer-theme-numix-icones/tuto-numix-2.png
[image2]: /img/posts/installer-theme-numix-icones/tuto-numix.png

<!--more-->
