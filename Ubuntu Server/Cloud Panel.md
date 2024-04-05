# Cloud Panel

## Objetivo
Instalar Cloud Panel en Ubuntu Server

## How to install CloudPanel on Ubuntu 22.04
[lilnk](https://najigram.com/2022/10/how-to-install-cloudpanel-on-ubuntu-22-04/).

Like other control panels CloudPanel is also a player in this segment providing clean interface to interact. It is limited, but it does the job pretty nicely with what it provides. It has a simple and easy to use dashboard. In this article we will go through how to install CloudPanel on your Ubuntu 22.04 LTS server.

Before we jump into the installation, here are the top features you may want to know.

- File Manager
- IP & Bot Blocking
- SSH/FTP
- Firewall
- Cron Jobs
- Vhost Editor
- Free Let’s Encrypt Certificates
- Cloudflare Integration
- User Management
- System resources usage graphs
- Multiple PHP versions
- MySQL and MariaDB support
- Node.js, Python support
- Nginx web server

But what it does not do is email management and anything related to mail. And that’s fine, because not everyone want to have that when the only thing they need is website management.

## Basic requirements:

Minimum 1 core
Minimum 2 GB RAM
10 GB of disk space

## Crear Virtual Box
Crear una maquina virtual con 25BG fijos NO virtuales.

## Installation
Visitar la pagina oficial de [Cloud Panel](https://www.cloudpanel.io/docs/v2/getting-started/other/).

Login to the server and update the system. Switch to root if not logged in as root and run.

Login a mi Virtual Machine (Latitude), correr en headless mode:
Configurar la maquina virtual para estar en modo *Network Bridge*, para saber el ip que se le ha asignado podemos buscarlo desde cliente con *angry iip scanner* o podemos entrar a la maquina y solicitar su ip con el comando: *ip a*.
`ssh enrique@192.168.0.10`

Switch to root:
 `sudo -i`
 
 Instalar curl:
`apt update && apt -y upgrade && apt -y install curl wget`

Run the following command to install CloudPanel. We will install it with MariaDB 10.11.
```
curl -sS https://installer.cloudpanel.io/ce/v2/install.sh -o install.sh; \
echo "85762db0edc00ce19a2cd5496d1627903e6198ad850bbbdefb2ceaa46bd20cbd install.sh" | \
sha256sum -c && sudo DB_ENGINE=MARIADB_10.11 bash install.sh
```

## Problema
Me sale un problema en virtualbox Error no hay suficiente esoacio en disco.
Voy a tratar de instalarlo en mi server fisico *optiplex755*.

*La solución:*
**How to make LV use all disk space in PV?**
[link](https://askubuntu.com/questions/1269493/how-to-make-lv-use-all-disk-space-in-pv).

I also used the default Ubuntu 20.04 install from ISO w/ lvm option selected. I had the same problem with the OS disk not occupying what I had allocated. Eddie's suggestion and the provided link did it for me. To summarize:
```
root@util:~# vgdisplay
root@util:~# lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
root@util:~# resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```




* * *
Listo!