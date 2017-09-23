+++
author = "arthur"
categories = ["linux"]
date = "2015-10-11"
description = ""
featured = "dns.png"
featuredalt = "Bannière de l'article"
featuredpath = "/img/posts/changer-dns-linux"
linktitle = ""
title = "Changer les DNS sous Linux"
type = "post"

+++

Bonjour, Aujourd’hui on va apprendre a modifié ses DNS. Il y a beaucoup d’avantage a avoir des serveurs DNS performants et fiable. Évitez les censures de l’état, plus de rapidité ou encore par soucis de sécurité.

Il faut donc éditer le fichier */etc/dhcp/dhclient.conf* pour y dé-commenter la ligne suivante :
```
prepend domain-name-servers <mettre les IPs des DNS voulu>
```
Vous pouvez en mettre jusqu’à 3 séparé par des virgules.

Je vous conseil ceux de [OpenNIC][opennic], les IPs apparaitrons en milieu de page après quelques secondes. Quand vous changez vos DNS assurez-vous que vous utilisez des DNS qui sont sûr. Vérifiez leurs réputations et leurs politique de confidentialité.

Puis il faut redémarrer le Gestionnaire de réseaux afin que les changements soit pris en compte :

```shell
service network-manager restart
```
Pour vérifier que tout c’est bien enregistrer vous pouvez afficher le contenu du *resolv.conf*

```shell
cat /etc/resolv.conf
```
Voila c’est tout pour ce tuto !

[opennic]: https://www.opennicproject.org/ "Site d'OpenNIC"
