# Linux Software

## Google Chrome

```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

## Adobe Reader

Use Google Chrome as PDF viewer

## PhpStorm

### PhpStorm Plugins

## VSCode

### VSCode Plugins

## Inkscape

```
sudo apt install inkscape -y
```

## XnView

*Alternative to IrfanView*

```
wget https://download.xnview.com/XnViewMP-linux-x64.deb
sudo dpkg -i XnViewMP-linux-x64.deb
```
<https://www.xnview.com/en/xnviewmp/>

## SQLYog

Downloads: <https://github.com/webyog/sqlyog-community/wiki/Downloads>

```
sudo apt-get install wine-stable
wget https://s3.amazonaws.com/SQLyog_Community/SQLyog+13.1.7/SQLyog-13.1.7-0.x64Community.exe
sudo wine SQLyog-13.1.7-0.x64Community.exe
```

## Microsoft Teams

Download: <https://www.microsoft.com/de-de/microsoft-teams/download-app#desktopAppDownloadregion>

```
wget https://packages.microsoft.com/repos/ms-teams/pool/main/t/teams/teams_1.4.00.13653_amd64.deb
sudo dpkg -i teams_1.4.00.13653_amd64.deb
```

## OBS Studio

```
sudo apt install ffmpeg
sudo add-apt-repository ppa:obsproject/obs-studio
sudo apt install obs-studio
```

## Pandoc

```
sudo apt install pandoc
```

## MiKTeX

Download: <https://miktex.org/download>

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys D6BC243565B2087BC3F897C9277A7293F59E4889
echo "deb http://miktex.org/download/ubuntu focal universe" | sudo tee /etc/apt/sources.list.d/miktex.list
sudo apt-get update
sudo apt-get install miktex
```

