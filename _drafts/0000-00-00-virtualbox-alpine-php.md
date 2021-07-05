# Alpine Setup

* Download the Virtual Alpine ISO, which is optimized for virtual systems, from the Alpine Download section.
  * https://www.alpinelinux.org/downloads/
  * https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/x86_64/alpine-virt-3.14.0-x86_64.iso
    
* Login as `root`, the password is empty.
* Change the keyboard layout: `setup-keymap de de`
* `setup-hostname -n alpine`
* `setup-interfaces` (eth0, dhcp)
* `/etc/init.d/networking --quiet start &`
* `ping example.com`  
* Add the community repositories: `setup-apkrepos`, enter 1
* Update the system: `apk update`  
* Install software:
  * `apk add apache2`


## Read more

* https://wiki.alpinelinux.org/wiki/Alpine_setup_scripts
* https://wiki.alpinelinux.org/wiki/Configure_Networking
