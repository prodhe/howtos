Wifi Hackbox
============

---

Grundinstallera
---------------

Ladda hem ArchLinux ARM

Kopiera till SD-kort med:

    diskutil list
    diskutil unmountDisk /dev/diskN
    sudo dd bs=8m if=archlinuxarm.img of=/dev/diskN

Starta rpi

    root / root

Byt lösenord på root

    passwd

Uppdatera systemet och installera vim

    pacman -Syu vim
    reboot

Avkommentera "sv_SE.UTF-8" i /etc/locale.gen och uppdatera sen med

    locale-gen

Definiera systemet till svenska och ändra tangentbordslayout, samt tidszon

    localectl set-locale LANG="sv_SE.UTF-8"
    localectl set-keymap sv-latin1
    timedatectl set-timezone Europe/Stockholm
    
Skapa /home med resterande diskutrymme

    fdisk /dev/mmcblk0
    n
    p
    <enter>
    <enter>
    <enter>
    w
    reboot

Formatera

    mkfs.ext4 /dev/mmcblk0p3
    e2label /dev/mmcblk0p3 home

Lägg in i /etc/fstab:

    /dev/mmcblk0p3  /home  ext4  defaults  0  0

Starta om

    reboot

---

Webserver och PHP
-----------------

Installera lighttpd och php

    pacman -S lighttpd php-cgi

Lägg till följande i /etc/lighttpd/lighttpd.conf

    server.modules += ("mod_fastcgi")
    index-file.names += ("index.php")
    fastcgi.server = ( 
        ".php" => (
            "localhost" => ( 
                "bin-path" => "/usr/bin/php-cgi",
                "socket" => "/tmp/php-fastcgi.sock",
                "broken-scriptfilename" => "enable",
                "max-procs" => 4,
                "bin-environment" => (
                    "PHP_FCGI_CHILDREN" => "1"
                )
            )
        )   
    )

Skapa en teststida

    echo "<?php phpinfo(); ?>" >> /srv/http/index.php

Aktivera vid start och starta om

    systemctl --system enable lighttpd
    reboot

Surfa till http://<rpi-IP>/ och se om det fungerar!

---