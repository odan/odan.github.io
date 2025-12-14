---
title: Fixing the libicu dependency error when installing .NET SDK 10 on Ubuntu
layout: post
comments: true
published: true
description: 
keywords: dotnet, dotnet10, Ubuntu, libicu78, libicu, wsl2, windows11, linux, windows
---

When installing **.NET SDK 10** on Ubuntu, you might encounter a dependency error related to **libicu**. This happens because the required ICU version is not yet available in the default Ubuntu repositories.

The problem: Today, when I tried to install .NET SDK 10 on Ubuntu I got the following error message:

```
sudo apt-get install -y dotnet-sdk-10.0
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:2 https://packages.microsoft.com/debian/13/prod trixie InRelease
Hit:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:5 https://ppa.launchpadcontent.net/ondrej/php/ubuntu noble InRelease
Hit:6 http://security.ubuntu.com/ubuntu noble-security InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 dotnet-runtime-deps-10.0 : Depends: libicu78 but it is not installable or
                                     libicu77 but it is not installable or
                                     libicu76 but it is not installable or
                                     libicu72 but it is not installable
E: Unable to correct problems, you have held broken packages.
```

In short: .NET 10 requires `libicu78`, but Ubuntu does not ship it yet.

The workaround is simple: install `libicu78` manually, then proceed with the .NET installation.

Download and install the missing deb package:

```
wget https://ftp.debian.org/debian/pool/main/i/icu/libicu78_78.1-2_amd64.deb
sudo dpkg -i libicu78_78.1-2_amd64.deb
```

Verify the installation:

```
dpkg -l | grep libicu
```

Expected result:

```
ii  libicu78:amd64                 78.1-2                                  amd64        International Components for Unicode
```

Now you can install .NET SDK 10:

Add the microsoft packages:

```
wget https://packages.microsoft.com/config/debian/13/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
```

Install the SDK:

```
sudo apt-get update
sudo apt-get install -y dotnet-sdk-10.0
```

Verify the SDK installation:

```
dotnet --list-sdks
```

Expected result:

```
10.0.101 [/usr/share/dotnet/sdk]
```

