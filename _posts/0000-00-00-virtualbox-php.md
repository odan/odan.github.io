---
title: VirtualBox - PHP Development environment
layout: post
comments: true
published: false
description:
keywords: php
---

## Requirements

* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

## Ubuntu Server Setup

* Download the ISO file for [Ubuntu Server](https://ubuntu.com/download/server) (Manual server installation)
* Open VirtualBox and create a new VM with the mounted ISO file
* Install Ubuntu Server
* After the installation, shutdown the VM

## Guest Extension Setup

* Start the Ubuntu guest virtual machine
* Login to the Ubuntu guest as a sudo user and install the packages required for building external kernel modules:
* `sudo apt update`
* `sudo apt install build-essential dkms linux-headers-$(uname -r)`
* From the virtual machine menu, click Devices -> “Insert Guest Additions CD Image”.
* Mount the CD-ROM:  
* `sudo mkdir -p /mnt/cdrom`
* `sudo mount /dev/cdrom /mnt/cdrom`
* Navigate to the directory and run the VBoxLinuxAdditions.run script to install the Guest Additions.
* `cd /mnt/cdrom`
* `sudo sh ./VBoxLinuxAdditions.run --nox11`
* Reboot the Ubuntu guest for changes to take effect:
`sudo shutdown -r now`

Read more: [How to Install VirtualBox Guest Additions on Ubuntu](https://linuxize.com/post/how-to-install-virtualbox-guest-additions-in-ubuntu/)

## Shared Folder Setup

* In the window of the running VM, you select `Device` > `Shared Folders`
* Add a new shared folder  
* Enter th local path `C:\xampp\htdocs`
* Folder name: `shared`
* Read only: No  
* Mount automatically: Yes
* Mount point: `/shared`
* Mount permanent: yes

## Network Adapter Setup

* Open the VM settings > Network
* Change "Adapter 1" type from "NAT" to "Network Bridge"
* Click at "Extended"
* Select the adapter: "Intel PRO/1000 MT Desktop..."
* Select the mode: "Allow all and host"
* Click on "Ok" to save all settings.
* Start the VM
* Login as super user with "sudo su"
* Enter `ifconfig` to find the local ip address, e.g. `192.168.0.172`

## Webserver Setup

* Start the VM
* Login as super user `sudo su`
* Copy this installation sh script to `/shared/setup.sh`

```sh
#!/usr/bin/env bash

apt-get update
apt-get install vim unzip apache2 -y
apt-get install mysql-client libmysqlclient-dev -y
apt-get install libapache2-mod-php7.4 php7.4 php7.4-mysql php7.4-sqlite -y
apt-get install php7.4-mbstring php7.4-curl php7.4-intl php7.4-gd php7.4-zip php7.4-bz2 -y
apt-get install php7.4-dom php7.4-xml php7.4-soap -y

a2enmod rewrite
a2enmod actions

sed -i '170,174 s/AllowOverride None/AllowOverride All/g' /etc/apache2/apache2.conf

rm -rf /var/www
mkdir /var/www
ln -s /shared/ /var/www/html

service apache2 restart

cd ~
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"
composer self-update
```

Set execute permission:

```
sudo chmod +x /shared/setup.sh
```

Run the setup script:

```
./shared/setup.sh
```

Add `vboxsf` to apache `www-data` group:

```
sudo usermod -a -G vboxsf www-data
```

Restart apache:

```
sudo service apache2 restart
```

On the guest VM open `http://192.168.0.172`in your browser to the hosted website.
Change the IP address as shown in `ifconfig`

