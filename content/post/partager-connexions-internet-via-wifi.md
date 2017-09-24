+++
author = "arthur"
categories = ["windows"]
date = "2015-10-21"
description = ""
featured = "partage-wifi.png"
featuredalt = "Bannière de l'article"
featuredpath = "/img/posts/partager-connexions-internet-via-wifi"
linktitle = ""
title = "Partager sa connexions internet via le wifi"
type = "post"

+++

Aujourd’hui on va partager sa connexion par Wifi. Imaginez que vous êtes chez un ami qui vous reçoit pour un jour ou deux dans son appartement étudiants. Dans les résidences étudiantes, il ne suffit pas d’avoir le mot de passe du wifi pour se connecter puisqu’un pc doit correspondre à un compte sur le portail captif. Cependant si un pc qui est connecté au portail via le compte de votre ami et que ce dit ami vous partage sa connexion via le wifi, alors le problème de connexion a internet est résolu.

<!--more-->

Ou sinon imaginons que votre ami a juste un câble Ethernet et pas de routeur sans fils à sa disposition.  

Lancer un invité de commande ou une fenêtre PowerShell en tant qu’administrateur. Puis pour définir un réseau wifi, il faut utiliser cette commande:

```dos
netsh wlan set hostednetwork mode=allow ssid=LeNomDeVotreWifi key=VotreMotDePasse!
```
Puis il faut lancer le réseaux Wifi que l’on a créé. Pour ceci utilisez cette commande :

```dos
netsh wlan start hostednetwork
```
Evidement, si vous voulez l’arretez c’est la même commande que au dessus mais en mettant stop:

```dos
netsh wlan stop hostednetwork
```
Ce n’est pas tout, si vous voulez que ça marche il faut partager votre connexion.

Allez dans le centre de réseaux et de partage, puis cliquez sur modifier les paramètres de la carte. Cliquez sur la carte en question et partagez la.

Si vous avez tous fais correctement et que ça ne marche pas essayez ceci :

```dos
netsh wlan show drivers
```

Et regarder en bas du ce que retourne la commande, il y a des chances que votre carte ne soit pas compatible.

Voila maintenant si vous héberger quelqu’un dans votre chambre d’étudiant lui aussi aura accès facilement à internet.
