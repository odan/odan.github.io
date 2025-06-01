---
title: PHP Development Setup on Windows with WSL2 and XDebug
layout: post
comments: true
published: false
description:
keywords: php, wsl, wsl2, docker, xampp
---

Installing Xdebug with a package manager is often the fastest way. 
Depending on your distribution, run the following command:

```
sudo apt update
sudo apt install php-xdebug
```

You can check whether it did by running php -v. 

If Xdebug shows up with a version number, 
than you're all set and you can configure Xdebug's other functions, 
such as Step Debugging or Profiling.

## Setup Step Debugging

The Xdebug's step debugger allows you to interactively walk through your 
code to debug control flow and examine data structures.

https://xdebug.org/docs/step_debug

Xdebug interacts with IDEs to provide step debugging functionality, and therefore you also need to configure an IDE that knows how to talk to Xdebug with the open DBGp protocol.

This protocol is supported by nearly every PHP IDE (including Visual Studio Code and PhpStorm), and also by text-based editors.

Ping the XDebug hostname:

```
echo "$(hostname).local"
```

Example output:

```
DESKTOP-ABCDEFG.local
```

Use your hostname with the `.local` prefix to configure Xdebug.

Open php.ini:

```
sudo vi /etc/php/8.4/apache2/php.ini
```

Add this configuration `[xdebug]` section. 

```
[xdebug]
xdebug.mode=debug
xdebug.client_host=DESKTOP-ABCDEFG.local
xdebug.start_with_request=trigger
```

Change the hostname accordingly.

Save the changes, by pressing `ESC`, then enter `:wq` and press `ENTER`.

Restart Apache:

```
sudo service apache2 restart
```

## Add "allow" rule to Windows firewall for WSL2 network

The XDebug client tries to connect to Windows machine from Linux which is blocked by default
because the WSL virtual NIC connection belongs to "Public" profile and almost all connections are forbidden.

Since the IP might change every time you restart your computer, it's easier to define an allo rule for the interface.

On Windows, open Powershell with admin rights and enter:

```
New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow
```

Read more:

* <https://github.com/microsoft/WSL/issues/4585#issuecomment-610061194>
* <https://superuser.com/a/1679774>

## Add a Remote server in PhpStorm

In PhpStorm open: `File -> Settings -> Languages & Frameworks -> PHP -> Servers`
Click: `Add New Server`

Enter

```
Name: WSL2
Host: localhost
Debugger: XDebug
```

Enable "Use path mappings" and link the absolute path on the server 
to the local project root directory only. 

Example

Local: c:\xampp\htdocs/example.com
Remote: /mnt/c/xampp/htdocs/example.com


Click on "Start Listening for PHP Debug Connections"


## Xdebug connection test

Test XDebug port

```
ping "$(hostname).local"
nc -zv "$(hostname).local" 9003

Connection to DESKTOP-EXAMPLE.local (172.26.128.1) 9003 port [tcp/*] succeeded!
```

## PhpStorm Xdebug Validation Report

This step is optional, but can be used to validate the Xdebug configuration for PhpStorm.

Download and extract the file `phpstorm_xdebug_validator.zip` from the Jetbrains server:

https://packages.jetbrains.team/files/p/ij/xdebug-validation-script/script/phpstorm_xdebug_validator.zip


In PhpStorm, click on "Start Listening for PHP Debug Connections"

Open the test page: http://localhost/xdebug/phpstorm_index.php

Test a breakpoint and the debugger should stop there.

