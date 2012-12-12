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

Reboot and relogin with <username>

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
	--nostorage
	--pppd
	--sudo

Now, connect and disconnect using:

	$ sudo ./sakis3g connect
	$ sudo ./sakis3g disconnect

To use the executable script locally, move it:

	$ sudo mv ./sakis3g /usr/bin/

To enable connection at startup, add the following to */etc/rc.local*

	/usr/bin/sakis3g connect

And for a clean exit, add to */etc/rc.local.shutdown*:

	/usr/bin/sakis3g disconnect


Make eth0 static
----------------

Disable dhcpcd on eth0 interface at startup

	$ sudo systemctl disable dhcpcd@eth0

Create the following file: */etc/conf.d/network*

	interface=eth0
	address=192.168.0.42
	netmask=24
	broadcast=192.168.0.255
	gateway=192.168.0.1

Create the following file: */etc/systemd/system/network.service*

	[Unit]
	Description=Network Connectivity
	Wants=network.target
	Before=network.target
	
	[Service]
	Type=oneshot
	RemainAfterExit=yes
	EnvironmentFile=/etc/conf.d/network
	ExecStart=/sbin/ip link set dev ${interface} up
	ExecStart=/sbin/ip addr add ${address}/${netmask} broadcast ${broadcast} dev ${interface}
	ExecStart=/sbin/ip route add default via ${gateway}
	ExecStop=/sbin/ip addr flush dev ${interface}
	ExecStop=/sbin/ip link set dev ${interface} down
	
	[Install]
	WantedBy=multi-user.target

Make it autostart as a static backup, in case the modem fails:

	$ sudo systemctl enable network


Bind the 3g-given IP to DNS
---------------------------

Register a free account at http://www.no-ip.org/ and choose a domain

Install the client

	$ sudo pacman -S noip

Configure the client (and answer no for the question of running something at successful update)

	$ sudo noip2 -C -Y

The configuration file */etc/no-ip2.conf* has now been created.

Add to startup

	$ sudo systemctl enable noip2


---

*Written by Prodhe*



