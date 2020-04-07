+++
author = "arthur"
categories = ["linux"]
date = "2020-04-07"
description = "Tutoriel montrant comment installer Kubernetes sur des VPS Ovh"
featured = "image1.png"
featuredalt = "Bannière de l'article"
featuredpath = "/img/posts/install-kubernetes-on-vps/"
linktitle = ""
title = "Installer Kubernetes"
type = "post"

+++

# Install Kubernetes


Bonjour aujourd'hui nous allons installer kubernetes. Ici nous le ferons avec Kubespray sur OVH.
Pour ce tutoriel j'ai trois **VPS SSD** premier prix avec **Ubuntu18.04**.
Il y a beaucoup de prerequis avant d'installer **Kubernetes**, comme s'assurrer que les VPS peuvent
communiquer et que ils sont dans le meme reseau privé, qu'ils ont un container runtime de fonctionnelle(rkt ou docker).


## Sommaire

1. Install OpenVPN
2. Install de Docker
3. Config de Kubespray
4. Test de l'instance

<!--more-->

# 1. Install de Openvpn

Pour installer Openvpn en un clin d'oeil j'utilise le script de [Nyr](https://github.com/Nyr/openvpn-install), sur le node1 uniquement:
```
wget https://git.io/vpn -O openvpn-install.sh && bash openvpn-install.sh
```

je commente dans le `/etc/openvpn/server/server.conf` la ligne `push "redirect-gateway def1 bypass-dhcp"`
j'ajoute la ligne `client-to-client` a la fin
puis je restart le service `systemctl restart openvpn`



je genere deux config client puisque un sera le serveur vpn et les deux autres les noeuds.
Niveau IP ca se passe comme ca :
* Node1 $rrar; 10.8.0.1
* Node2 $rrar; 10.8.0.2
* Node3 $rrar; 10.8.0.3


je **SCP** mes config sur les serveur et je les renomme. Afin que mes noeuds se connecte automatiquement j'installe et configure le client openvpn puis j'edite decommente la ligne `AUTOSTART="all"` dans le fichier `/etc/default/openvpn`  et je reboot afin de tester que tout fonctionne pour le mieux.

```
mv node[2/3].ovpn /etc/openvpn/client.conf
apt update && apt install openvpn
vim /etc/default/openvpn
systemctl enable --now openvpn@client
systemctl reboot
```
# 2. Install de docker

Basiquement j'install docker sur chacun de mes noeuds: 

```
apt-get update
apt-get remove docker docker-engine docker.io containerd runc
apt-get update
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

apt-get update
apt-get install docker-ce-cli containerd.io
apt-get install docker-ce
docker run hello-world
```


# 3. Config de kubespray

je scp ma clefs github sur le server, je clone le repo, je change de branche sur celle de mon choix. Puis je configure l'installation et j'installe les derniers packets necessaire. La configuration de kubespray se fait en suivant betement le readme.
```
pip3 install ansible==2.7.16
git clone git@github.com:kubernetes-sigs/kubespray.git
cd kubespray
git checkout release-2.9
pip3 install -r requirements.txt
pip3 install ansible==2.7.16
cp -rfp inventory/sample inventory/mycluster
declare -a IPS=(10.8.0.1 10.8.0.2 10.8.0.3 10.8.0.4)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```
Il y a des flags Ansible qui peuvent etre utile : `--flush-cache` ou encore `--ask-become-pass`


# 4. Test de l'instance

donc on peut s'amuser a lancer quelques deployment
```
kubectl create deployment test --image=gcr.io/hello-minikube-zero-install/hello-node
kubectl expose deployment test --type=LoadBalancer --port=8080
kubectl scale deployment test --replicas=10
kubectl delete deployment test
```


