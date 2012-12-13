Raspberry Pi, Arch Linux and 3G modem
=====================================

Short guide to install a Huawei E173 3G modem on an Arch Linux installation on Raspberry Pi.


Setup
-----

- Download from rpi homepage.
- Plug in SD card

On Mac OS X

	$ diskutil list
	$ diskutil unmountDisk /dev/disk<#>
	$ dd bs=1m if=downloaded-arch-file.img of=/dev/disk<#>


First boot
----------

### Update and create user

Up-to-date the system

	# pacman -Syyu

Install user and sudo and relogin

	# useradd -m -g users -G wheel <username>
	# passwd <username>
	# pacman -S sudo

Add wheel group to sudo by uncommenting that line

	# visudo

### Locales and timezone

Uncomment preferred locale in */etc/locale.gen* and update

	# locale-gen

Define system globals

	# localectl set-locale LANG="sv_SE.UTF-8"
	# localectl set-keymap sv-latin1
	# timedatectl set-timezone Europe/Stockholm

Reboot and relogin with the above username

	# shutdown -r now


Configure 3G modem
------------------

### Install needed packages

Install ppp and USB_ModeSwitch (to handle those ZeroCD-things that modems usually have...)

	$ sudo pacman -S ppp usb_modeswitch

Sakis3G binary free version

	$ wget http://www.sakis3g.org/versions/latest/binary-free/sakis3g.gz
	$ gunzip sakis3g.gz
	$ chmod +x sakis3g

Shutdown the R-Pi and plug in the modem

	$ sudo shutdown -h now

### Configure modem

Start the device with the usb dongle attached, login and run

	$ sudo ./sakis3g --interactive

Add the variables mentioned in the scripts to */etc/sakis3g.conf*

	OTHER="USBMODEM"
	USBMODEM="12d1:1436"
	USBINTERFACE="0"
	APN="online.telia.se"

Add some more configurations (read about it on wiki.sakis3g.org)

	--console
	--googledns
	--pppd
	--sudo

Now, connect and disconnect using:

	$ sudo ./sakis3g connect
	$ sudo ./sakis3g disconnect

To use the executable script globally, move it:

	$ sudo mv ./sakis3g /usr/bin/


Handle network cable
--------------------

Disable dhcpcd on eth0 interface at startup

	$ sudo systemctl disable dhcpcd@eth0

Install netcfg and ifplugd (required for net-auto-wired)

	$ sudo pacman -S netcfg ifplugd

All network profiles are stored in */etc/network.d/*.

Copy the standard wired examples (ethernet-dhcp and ethernet-static) and change as you wish:

	$ cd /etc/network.d
	$ sudo cp examples/ethernet-static mystaticnet
	$ sudo cp examples/ethernet-dhcp mydhcpnet

Activate the profiles in */etc/conf.d/netcfg*

	NETWORKS=(mydhcpnet mystaticnet)

Activate the service

	$ sudo systemctl enable net-auto-wired

The net-auto-wired will act if you plug the cable in, and will always start with the DHCP


Bind the 3g-given IP to DNS
---------------------------

Register a free account at http://www.no-ip.org/ and choose a domain

Install the client

	$ sudo pacman -S noip

Configure the client, bind to PPP and answer no for the question of running something at successful update

	$ sudo noip2 -C -Y

The configuration file */etc/no-ip2.conf* has now been created.

Add to startup

	$ sudo systemctl enable noip2


Autoconnect the 3G modem on startup
-----------------------------------

To enable connection at startup, create the following file */etc/systemd/system/sakis3g.service*

	[Unit]
	Description=Sakis3G
	
	[Service]
	ExecStart=/usr/bin/sakis3g connect
	ExecStop=/usr/bin/sakis3g disconnect
	
	[Install]
	WantedBy=multi-user.target

Enable the autostart:

	$ sudo systemctl enable sakis3g


---

*Written by Prodhe*


