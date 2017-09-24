+++
author = "arthur"
categories = ["linux", "virtualization"]
date = "2017-09-24"
description = ""
draft = "true"
featured = ""
featuredalt = ""
featuredpath = "/img/posts/install-kvm-to-arch-linux"
linktitle = ""
title = "Install KVM to Arch Linux"
type = "post"

+++

# Install KVM to Arch Linux

KVM (Kernel-based Virtualization Machine )is a powerful virtualization tool that enable you to visualize multiple OS without performance loss, without the need to install a virtualization specific OS like XEN, or a GUI(Graphical User Interface) to your server. As always on GNU/Linux no need of spending half of your paycheck here, everything is doable on your home server.

## Check Hardware compatibility

Minimal Hardware requirements are that the processor need to have intel's VT or AMD-V enabled in order for it to function. You may have to into the BIOS and enable a few options. To check if our cpu has what it needs type the following command.

```shell
egrep --color=auto 'vmx|svm|0xc0f' /proc/cpuinfo
```

## Install the needed package

```shell
pacman -S openbsd-netcat qemu libvirt dnsmasq qemu-arch-extra firewalld qemu-block-iscsi qemu-block-rbd qemu-block-gluster qemu-guest-agent
vim /etc/libvirt/qemu.conf
```

remove the previous group line and add :

```shell
user = "root"
group = "root"
```

## Start the Services

```shell
systemctl restart firewalld
systemctl restart libvirtd
```

## Make the services start at boot

```shell
systemctl enable firewalld
systemctl enable libvirtd
```

And your done.

I would recommend using a tool called virt-manager on your computer, but that is for next time
