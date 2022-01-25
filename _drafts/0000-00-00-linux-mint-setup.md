# Linux Mint XFCE Setup for PHP developers

## Linux Mint Setup

* Download the latest ISO from [the Linux mint website](https://blog.linuxmint.com/?p=4013)
* Copy the ISO file with [E](https://www.balena.io/etcher/) to a USB stick and install it.

## Install Virtualbox guest extension

* From the virtual machine menu, click `Devices` -> `Insert Guest Additions CD Image`.
* Open the terminal and enter:
* `cd /media/user/VBox_GAs_6.1.26/`
* `sudo sh ./VBoxLinuxAdditions.run --nox11`  
* Reboot the VM for changes to take effect: `sudo shutdown -r now`

## Uninstall Software

```
# LibreOffice
sudo apt remove --purge -y libreoffice*
sudo apt remove --purge -y libjuh*

# Remove all LibreOffice start menu icons
find ~/.local/share/applications/ -type f -name '*libreoffice*.desktop' -delete

# Gimp
sudo apt remove --purge -y gimp

# Thunderbird
sudo apt remove --purge -y thunderbird

sudo apt clean
sudo apt autoremove
```

## System Updates

Update all Packages

```
sudo apt update
```

Update only security updates

```
sudo apt install unattended-upgrades
```

Upgrade the Linux system

```
sudo apt upgrade
```

Upgrade the Linux distribution

```
sudo apt dist-upgrade
```

## BIOS

```
sudo apt install intel-microcode
```

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

Change keyboard shortcut for `xflock4` to `Shift+Ctrl+L`

### PhpStorm Themes

[Photon color Theme for PhpStorm](https://github.com/brendt/phpstorm-photon-theme)

```
su user
cd ~/Downloads
wget https://raw.githubusercontent.com/brendt/phpstorm-photon-theme/master/Photon%20-%20Dark.icls -O Photon-Dark.icls
wget https://raw.githubusercontent.com/brendt/phpstorm-photon-theme/master/Photon%20-%20Light.icls -O Photon-Light.icls
```

[Github 3 Theme for PhpStorm](https://plugins.jetbrains.com/plugin/12271-github-3-color-scheme)

## Install Google Chrome

To install Google Chrome, run the following:

```
cd ~/Downloads
sudo apt install libxss1 libappindicator1 libindicator7
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb -y
```

If error messages pop up after running the command sudo apt install ./google-chrome*.deb then run the command

```
sudo apt install -f.
```

If you want to install chromium instead, run:

```
sudo snap install chromium
```

Or with apt-get:

```
sudo apt install chromium-browser chromium-browser-l10n chromium-codecs-ffmpeg 
```

## Wine

```
sudo apt install wine-stable 
```

## Install Apache and PHP with XDebug

...

## Fonts

**Microsoft Fonts, Arial, Verdana etc**

```
sudo apt install ttf-mscorefonts-installer rar unrar libavcodec-extra gstreamer1.0-libav gstreamer1.0-plugins-ugly gstreamer1.0-vaapi
```

Segoe UI font, however, is not part of the ttf-mscorerfonts package.
To install the full pack of Segoe UI fonts run:

```
sudo su
mkdir -p /usr/share/fonts/truetype/
cd /usr/share/fonts/truetype/
wget -q https://github.com/martinring/clide/blob/master/doc/fonts/segoeui.ttf?raw=true -O segoeui.ttf
wget -q https://github.com/martinring/clide/blob/master/doc/fonts/segoeuib.ttf?raw=true -O segoeuib.ttf
wget -q https://github.com/martinring/clide/blob/master/doc/fonts/segoeuib.ttf?raw=true -O segoeuii.ttf
wget -q https://github.com/martinring/clide/blob/master/doc/fonts/segoeuiz.ttf?raw=true -O segoeuiz.ttf
wget -q https://github.com/martinring/clide/blob/master/doc/fonts/segoeuil.ttf?raw=true -O segoeuil.ttf
wget -q https://github.com/martinring/clide/blob/master/doc/fonts/seguili.ttf?raw=true -O seguili.ttf
wget -q https://github.com/martinring/clide/blob/master/doc/fonts/segoeuisl.ttf?raw=true -O segoeuisl.ttf
wget -q https://github.com/martinring/clide/blob/master/doc/fonts/seguisli.ttf?raw=true -O seguisli.ttf
wget -q https://github.com/martinring/clide/blob/master/doc/fonts/seguisb.ttf?raw=true -O seguisb.ttf
wget -q https://github.com/martinring/clide/blob/master/doc/fonts/seguisbi.ttf?raw=true -O seguisbi.ttf
fc-cache -frv
```

**IBM Plex Mono** *for PhpStorm*

```
sudo su
mkdir -p /usr/share/fonts/truetype/
cd /usr/share/fonts/truetype/
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-Bold.ttf -O IBMPlexMono-Bold.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-BoldItalic.ttf -O IBMPlexMono-BoldItalic.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-ExtraLight.ttf -O IBMPlexMono-ExtraLight.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-ExtraLightItalic.ttf -O IBMPlexMono-ExtraLightItalic.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-Italic.ttf -O IBMPlexMono-Italic.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-Light.ttf -O IBMPlexMono-Light.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-LightItalic.ttf -O IBMPlexMono-LightItalic.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-Medium.ttf -O IBMPlexMono-Medium.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-MediumItalic.ttf -O IBMPlexMono-MediumItalic.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-Regular.ttf -O IBMPlexMono-Regular.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-SemiBold.ttf -O IBMPlexMono-SemiBold.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-SemiBoldItalic.ttf -O IBMPlexMono-SemiBoldItalic.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-Text.ttf -O IBMPlexMono-Text.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-TextItalic.ttf -O IBMPlexMono-TextItalic.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-Thin.ttf -O IBMPlexMono-Thin.ttf
wget -q https://raw.githubusercontent.com/IBM/plex/master/IBM-Plex-Mono/fonts/complete/ttf/IBMPlexMono-ThinItalic.ttf -O IBMPlexMono-ThinItalic.ttf
fc-cache -frv
```


