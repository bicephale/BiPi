# BiPi Documentation

This is the official documentation for the Bicephale 3D Printer Raspberry Pi Image.

## Network

Netconnectd seems to override (or disable) native DHCP daemon, need to edit **/etc/network/interfaces** and set eth0 to

    iface eth0 inet dhcp

    sudo nano /etc/dhcp/dhclient.conf

Then set the **timeout** value to something convenient.



BiPi basic cooking
===================


If you want to cook your very own version of **BiPi** follow the basic steps below wich are preliminary to an up and running system.

----------

Minibian Setup
-------------

First, you'll have to setup Minibian on your Raspberry Pi 2. Download & install it to the SD card.

Follow this guide for the initial setup http://www.htpcguides.com/lightweight-raspbian-distro-minibian-initial-setup/

During the first boot, you'll have to do some basic steps in the **raspi-config** utility

> **Note:**

> - Expand filesystem
> - Enable camera
> - Enable boot to console
> - Set GPU memory to **128mb** (otherwise, camera won't work)
> - Change hostname to bicephale

Then change the classic Pi username with the command below. Be carefull, you'll have to do this with a screen, it won't work over SSH.

    exec sudo -s
    cd /
    usermod -l bmk -d /home/bmk -m pi
    reboot

Change the default password, by purpose, we'll use a convention everywhere, username **bmk** password **bicephale** Use the command below to change the password :

    passwd

OctoPrint setup
-------------

Please setup Octoprint using this guide https://github.com/foosel/OctoPrint/wiki/Setup-on-a-Raspberry-Pi-running-Raspbian be carefull to use the Bicephale's OctoPrint fork repo instead of the classic one.

In the part relative to setup scripts please change in **/etc/default/octoprint** the user to **bmk** (instead of Pi) and executable path to **DAEMON=/home/bmk/OctoPrint/venv/bin/octoprint**

    sudo nano /etc/default/octoprint

Netconnectd setup
-------------

Install netconnectd following this guide https://github.com/foosel/netconnectd

## Setup

### Prepare the system

Install the hostapd, dnsmasq, logrotate and rfkill packages:

    sudo apt-get install hostapd dnsmasq logrotate rfkill

----

**Note for people updating**: Netconnectd now depends on the ``rfkill`` tool to be installed on the target system as
well, the above package installation instructions have since been updated to reflect this.

----

We don't want neither `hostapd` nor `dnsmasq` to automatically startup, so make sure their automatic start on boot is 
disabled:

    sudo update-rc.d -f hostapd remove
    sudo update-rc.d -f dnsmasq remove

You can verify that this worked by checking that there are no files left in `/etc/rc*.d` referencing those two services,
so the following to commands should return `0`:

    ls /etc/rc*.d | grep hostapd | wc -l
    ls /etc/rc*.d | grep dnsmasq | wc -l

If you are running NetworkManager (default for Ubuntu or other desktop linux distributions, usually not the case for 
Raspbian), make sure to disable its own `dnsmasq` by editing `/etc/NetworkManager/NetworkManager.conf` and commenting
out the line that says `dns=dnsmasq`, it should look something like this afterwards (note the `#` in front of the
`dns` line):

    [main]
    plugins=ifupdown,keyfile,ofono
    #dns=dnsmasq
    
    no-auto-default=00:22:68:1F:83:AF,
    
    [ifupdown]
    managed=false

You'll also need to modify `/etc/dhcp/dhclient.conf` to include a timeout setting, e.g.

    timeout 60;

Otherwise -- due to a limitation of how Debian/Ubuntu currently parses Wifi configurations in `/etc/network/interfaces` 
-- netconnectd won't be able to detect when it couldn't connect to your configured local wifi and will never start the 
access point mode. The value above will mean that it will take a maximum of 60sec before netconnectd will be notified 
by the system that the connection was unsuccessful -- you might want to lower that value even more but keep in mind that 
your wifi's DHCP server has to respond within that timeout for the connection to be considered successful.

Once it's done, please update HOSTAPD from Adafruit (https://learn.adafruit.com/setting-up-a-raspberry-pi-as-a-wifi-access-point/install-software) with the steps below (**depending of your WiFi driver !**) :

    wget http://adafruit-download.s3.amazonaws.com/adafruit_hostapd_14128.zip
    unzip adafruit_hostapd_14128.zip
    sudo mv /usr/sbin/hostapd /usr/sbin/hostapd.ORIG
    sudo mv hostapd /usr/sbin
    sudo chmod 755 /usr/sbin/hostapd

Then, to test if it's working, create hostapd config file in /tmp/hostapd.conf (JUST FOR TESTING PURPOSES)

    interface=wlan0
    driver=rtl871xdrv
    ssid=Bicephale
    channel=3
    wpa=3
    wpa_passphrase=bicephale              
    wpa_key_mgmt=WPA-PSK
    wpa_pairwise=TKIP CCMP
    rsn_pairwise=CCMP

And run the command below :

    sudo /usr/sbin/hostapd /tmp/hostapd.conf

You should have an AP up and running. Then, install the netconnect daemon

https://github.com/foosel/netconnectd

### Install netconnectd

It's finally time to install `netconnectd`:

    cd
    git clone https://github.com/foosel/netconnectd
    cd netconnectd
    sudo python setup.py install
    sudo python setup.py install_extras

Modify `/etc/netconnectd.yaml` as necessary:
 
  * Change the passphrase/psk for your access point
  * If necessary change the interface names of your wifi and wired network interfaces dont forget to change the wifi driver
  * If your machine is **not** running NetworkManager, set `wifi > free` to `false`
  * if you **don't** want to reset the wifi interface in case of any detected errors on the driver level, set
    `wifi > kill` to `false`
 
Last, start netconnectd:

    sudo service netconnectd start

Verify that the logfile looks ok-ish:

    less /var/log/netconnectd.log

and that it's indeed running (error handling of the start up script still needs to be improved):

    netconnectcli status

Congratulations, `netconnectd` is now running and should detect when you don't have any connection available, starting the AP mode to change that.

If everything looks alright, configure the service so that it starts at boot up:

    sudo update-rc.d netconnectd defaults 98


Then install plugin https://github.com/OctoPrint/OctoPrint-Netconnectd and make sure that you previously source /venv/bin/activate before installing (to be in the same OctoPrint's Python environement)

Check /etc/network/interfaces and re enable eth0 dhcp

Misc
-------------

Autmatic startup MJPStreamer https://github.com/foosel/OctoPrint/issues/441

PySerial 250000 baudrate https://github.com/foosel/OctoPrint/wiki/OctoPrint-support-for-250000-baud-rate-on-Raspbian

Custom boot screen
http://www.raspberry-projects.com/pi/pi-operating-systems/raspbian/custom-boot-up-screen

Custom splash screen instructions (alternate) http://www.edv-huber.com/index.php/problemloesungen/15-custom-splash-screen-for-raspberry-pi-raspbian

## Licence

Unless otherwise specified, everything in this repository is covered by the following licence:

[![Creative Commons Attribution-ShareAlike 4.0 International](https://licensebuttons.net/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

***Raspberry Pi Documentation*** by the [Raspberry Pi Foundation](https://www.raspberrypi.org/) is licensed under a [Creative Commons Attribution 4.0 International Licence](http://creativecommons.org/licenses/by-sa/4.0/).

Based on a work at https://github.com/bicephale/documentation
