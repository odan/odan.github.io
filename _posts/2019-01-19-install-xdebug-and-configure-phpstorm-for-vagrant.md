---
title: Using Xdebug with Vagrant and PHPStorm
layout: post
comments: true
published: false
description: 
keywords: PhpStorm, Xdebug, Vagrant, php, debugger, linux
---


## Intro

I'm already aware that people use dockers these days, but I was dissatisfied with the performance on Mac. Even under Windows, many hacks and additional tools (docker-sync) are needed to work with Docker, and yet I wasn't satisfied. For these reasons, I'm back at Vagrant and describe here how to debug with PhpStorm PHP applications that run in a (good old) Vagrant box.

## Install xdebug

I'm using [Ubuntu, Apache and PHP 7.2 vagrant box setup](https://odan.github.io/2018/10/09/vagrant-with-ubuntu-18-04-setup.html).

If you have already setup another Vagrant box, please install and configure xdebug with this commands:

```bash
# install the php xdebug extension
sudo apt-get install php-xdebug

# add the xdebug settings to php.ini
sudo su
echo 'xdebug.remote_port=9000' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.remote_enable=true' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.remote_connect_back=true' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.remote_autostart=on' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.remote_host=' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.max_nesting_level=1000' >> /etc/php/7.2/apache2/php.ini
echo 'xdebug.idekey=PHPSTORM' >> /etc/php/7.2/apache2/php.ini

# restart apache
sudo service apache2 restart
```

Start your vagrant machine with: `vagrant up`.

## PhpStorm Configuration

* Open PhpStorm
* Click "Add configuration" (or Edit Configuration)

![image](https://user-images.githubusercontent.com/781074/51430761-dedb7900-1c1f-11e9-85b9-d45a0752cfa3.png)

* Click the `+` Button, then choose "PHP Remote Debug"
* Click the `...` Button and add a new Host `localhost`, and HTTP-Port: `8080` or `8888` (depends on your setup)
* Enable the `Use path mappings` checkbox
* Enter the absolute path on the server e.g. `/vagrant/my-project`.

![image](https://user-images.githubusercontent.com/781074/57547122-2ff1b300-735e-11e9-8d91-94315b310227.png)

* Enter the IDE key (session id): `PHPSTORM`
* Click OK to save the new settings
* Select the new configuration from the configuration menu
* Start the Xdebug session

![image](https://user-images.githubusercontent.com/781074/51430854-1696f080-1c21-11e9-8b62-f409878acb0b.png)

## Debugging

* Open the browser
* Add the xdebug cookie with the [PhpStorm Xdebug bookmarklets](https://www.jetbrains.com/phpstorm/marklets/)
* Add a breakpoint

![image](https://user-images.githubusercontent.com/781074/51430916-f3b90c00-1c21-11e9-8d06-b1a97aee98f0.png)

* Start your program now. As soon as your program reaches the marked position, the debugger should stop at this point.



