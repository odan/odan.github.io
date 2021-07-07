# Linux Mint XFCE Setup for PHP developers

## Linux Mint Setup

* Download the latest ISO file from [the Linux mint website](https://blog.linuxmint.com/?p=4013)
* Copy the ISO file with [E](https://www.balena.io/etcher/) to a USB stick and install it.

## Install Virtualbox guest extension

* From the virtual machine menu, click `Devices` -> `Insert Guest Additions CD Image`.
* Open the terminal and enter:
* `cd /media/user/VBox_GAs_6.1.22/`
* `sudo sh ./VBoxLinuxAdditions.run --nox11`  
* Reboot the VM for changes to take effect: `sudo shutdown -r now`

## Enable snapd

On Linux Mint 20, /etc/apt/preferences.d/nosnap.pref needs to 
be removed before Snap can be installed. This can be accomplished from the command line:

```
sudo rm /etc/apt/preferences.d/nosnap.pref
sudo apt update
sudo apt install snapd
```

## Install PhpStorm

PhpStorm is available in the Software Manager, but the Setup will not work.
For this reason I prefer to install it manually.
To install PhpStorm, simply use the following command:

```
sudo snap install phpstorm --classic
```

## Install Google Chrome

To install Google Chrome, run the following:

```
cd ~/Downloads
sudo apt-get install libxss1 libappindicator1 libindicator7
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb
```

If error messages pop up after running the command sudo apt install ./google-chrome*.deb then run the command

```
sudo apt-get install -f.
```

If you want to install chromium instead, run:

```
sudo snap install chromium
```

Or with apt-get:

```
sudo apt-get install chromium-browser chromium-browser-l10n chromium-codecs-ffmpeg 
```

## Install Apache and PHP with XDebug

...