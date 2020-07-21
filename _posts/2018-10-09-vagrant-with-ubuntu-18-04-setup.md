---
title: Vagrant with Ubuntu 18.04 Setup
layout: post
comments: true
published: false
description: 
keywords: 
---

This tutorial shows you how to install a development LAMP stack with Vagrant.
All required components (Apache, MySQL and PHP) will be installed from the official repositories.

## Requirements

* [Vagrant](https://www.vagrantup.com/downloads.html)

### Setup

Create a file called `Vagrantfile` in `c:\xampp\htdocs`

```vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.provision :shell, path: "bootstrap.sh"
  config.vm.network "forwarded_port", guest: 80, host: 8765
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.customize ['modifyvm', :id, '--cableconnected1', 'on']
  end  
end
```

* Create a file: `bootstrap.sh` in `c:\xampp\htdocs`.

```sh
#!/usr/bin/env bash

apt-get update

# unzip is for composer
apt-get install vim unzip  -y

apt-get install apache2 -y

apt-get install mysql-client libmysqlclient-dev -y
apt-get install libapache2-mod-php7.2 php7.2 php7.2-mysql php7.2-sqlite -y
apt-get install php7.2-mbstring php7.2-curl php7.2-intl php7.2-gd php7.2-zip php7.2-bz2 -y
apt-get install php7.2-dom php7.2-xml php7.2-soap -y
apt-get install --reinstall ca-certificates -y

# install the php xdebug extension (only for dev servers!)
apt-get install php-xdebug -y

# add xdebug settings to php.ini
echo 'xdebug.remote_port=9000' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.remote_enable=true' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.remote_connect_back=true' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.remote_autostart=on' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.remote_host=' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.max_nesting_level=1000' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.idekey=PHPSTORM' >> /etc/php/7.2/apache2/php.ini

# Enable apache mod_rewrite
a2enmod rewrite
a2enmod actions

# Change AllowOverride from None to All (between line 170 and 174)
sed -i '170,174 s/AllowOverride None/AllowOverride All/g' /etc/apache2/apache2.conf

# Start the webserver
service apache2 restart

# Install MySQL (optional)
# apt-get install mysql-server -y

# Change mysql root password
# service mysql start
# mysql -u root --password="" -e "update mysql.user set authentication_string=password(''), plugin='mysql_native_password' where user='root';"
# mysql -u root --password="" -e "flush privileges;"

# Install composer
cd ~
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"
composer self-update

# Create a symlink
rm -rf /var/www
mkdir /var/www
ln -s /vagrant/ /var/www/html
```

**Note** On a production server you should run `sudo mysql_secure_installation` to improve the security of your MySQL installation.

Read more:

* <https://dev.mysql.com/doc/refman/5.7/en/mysql-secure-installation.html>
* <https://mariadb.com/kb/en/library/mysql_secure_installation/>
* [How to setup a restricted SFTP server on Ubuntu?](https://askubuntu.com/questions/420652/how-to-setup-a-restricted-sftp-server-on-ubuntu)
* [Correct owner and permissions of apache "var/www/html"](https://odan.github.io/2019/02/17/correct-owner-and-permissions-of-var-www-html.html)
* [Using Xdebug with Vagrant and PHPStorm](https://odan.github.io/2019/01/19/install-xdebug-and-configure-phpstorm-for-vagrant.html)

### Up and Running

To start the vagrant box under windows, run:

```
cd c:\xampp\htdocs
vagrant up
```

* Open http://localhost:8765
