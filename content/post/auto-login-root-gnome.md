+++
author = "remy"
categories = ["linux", "gnome"]
date = "2014-08-31"
description = "Tutoriel pour activer la connexion automatique en tant que root avec Gnome 3"
featured = "gnome-logo.png"
featuredalt = "Image seedbox"
featuredpath = "/img/posts/auto-login-root-gnome"
linktitle = ""
title = "Se connecter automatiquement en root avec Gnome"
type = "post"

+++


Lorsque l’on utilise beaucoup de machines virtuelles sous [Debian][debian] (avec [Gnome][gnome]), on est très vite lassé de devoir se connecter avec un utilisateur pour ensuite lancer le terminal en root (où là aussi, il faut entrer le mot de passe).

Donc voici la solution pour que la connexion en root soit automatique :

1. Éditez le fichier /etc/pam.d/gdm3-autologin (Il contient les règles de connexion de gnome dont la règle interdisant le login en root)
2. Commentez la ligne interdisant le login en root : `auth    required    pam_succeed_if.so user != root quiet_success`
3. Éditez ensuite le fichier de configuration du démon de gnome 3 : */etc/gdm3/daemon.conf*
4. Décommentez les lignes `AutomaticLoginEnable` et `AutomaticLogin` et définissez la valeur de `AutomaticLogin` sur root :
```
AutomaticLoginEnable=True
AutomaticLogin=root
```
5. Redémarrez et le tour est joué : vous serez automatiquement connecté en root.

Si vous obtenez un chargement infini lors de la connexion, passez en mode console (`ctrl+alt+f1`), connectez-vous en root et vérifiez que vous avez bien édité les règles de connexion de gnome.

[debian]: https://www.debian.org/index.fr.html "Site officiel de Debian"
[gnome]: http://www.gnome.org/ "Site officiel de Gnome"
