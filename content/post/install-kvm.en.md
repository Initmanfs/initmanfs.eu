+++
author = "arthur"
categories = ["linux"]
date = "2018-01-05"
description = "Recipe for KVM installation"
featured = "Kvm.png"
featuredalt = "Banner of the article"
featuredpath = "/img/posts/install-kvm"
linktitle = ""
title = "Install KVM to ArchLinux"
type = "post"

+++
# Install KVM to Arch Linux

So KVM is a very efficient type 1 hypervisor, it is free and it comes with all the power of the linux community and of Open Source Software.
Arch is a very lightweight distro and very customizeable, people tend to prefer "stabler distro" such as debian or centos but in fact Arch 
is pretty stable and has the latest cutting edges technology, and no big update only rolling update no release updates.

## Check Hardware compatibility

Ssh into your server so you verify that the CPU of your server can run Virtual Machines.

```bash
egrep --color=auto 'vmx|svm|0xc0f' /proc/cpuinfo
```


## Install the needed packages

```bash
pacman -S openbsd-netcat  qemu libvirt dnsmasq qemu-arch-extra firewalld qemu-block-iscsi qemu-block-rbd qemu-block-gluster qemu-guest-agent
```

Then edit that file so that libvirt daemon is launch as root.

```bash
vim /etc/libvirt/qemu.conf
```
Add :

```
user = "root"
group = "root"
```

Remove the previous group line they are irrelevant.


## Start the Services

Use systemd command line controller called systemctl to restart the 2 services.

```bash
systemctl restart firewall libvirtd
```

## Make the services start at boot

```bash
systemctl enable firewalld libvirtd
```

## Connection to Kvm over ssh

We are now making your local computer connect to kvm hypervisor via ssh.

### SSH config

Make a ssh folder :

```bash
mkdir ~/.ssh
```

Then create the config file.

```bash
vim ~/.ssh/config
```

Then you can edit and add for example :

```
Host The_Name # EMail, kubernetes, whatever..
    Hostname IP_ADDRESS
    Port ThePortNumber
    User TheUser
    IdentityFile ~/.ssh/YourPrivateKey
```

For more information: http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/

### Virt-Manager

Download and install virt-manager, it is in the repository.

Start Virt-Manager and go File > Add Connection

![tuto-kvm-arch][image1]

Then fill in with what you put in ~/.ssh/config

![tuto-kvm-arch][image2]

And then you can create Virtual machine on your server, just download ISOs to "/var/lib/libvirt/".

[image1]: /img/posts/install-kvm/image1.png "Virtual Machine Manager interface"
[image2]: /img/posts/install-kvm/image2.png "Add Connection to Virtual Machine Manager"
