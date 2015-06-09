# BiPi Documentation

This is the official documentation for the Bicephale 3D Printer Raspberry Pi Image.

## Install

### OSX

    gzip -dc /path_to_image/bipi-xxx.xxx.gz | sudo dd of=/dev/rdiskID bs=1m

## Backup

### OSX

    sudo dd if=/dev/rdiskID bs=1m | gzip > /path_to_image/bipi-xxx.xxx.gz

## Infos

Login and password are system wide

**Login :** bmk
**Password :** bicephale

## Network

Netconnectd seems to override (or disable) native DHCP daemon, need to edit **/etc/network/interfaces** and set eth0 to

    iface eth0 inet dhcp

    sudo nano /etc/dhcp/dhclient.conf

Then set the **timeout** value to something convenient.


## Licence

Unless otherwise specified, everything in this repository is covered by the following licence:

[![Creative Commons Attribution-ShareAlike 4.0 International](https://licensebuttons.net/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

***Raspberry Pi Documentation*** by the [Raspberry Pi Foundation](https://www.raspberrypi.org/) is licensed under a [Creative Commons Attribution 4.0 International Licence](http://creativecommons.org/licenses/by-sa/4.0/).

Based on a work at https://github.com/bicephale/documentation
