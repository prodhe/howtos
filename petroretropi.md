PetroRetroPi
============

Installera
----------

- Ladda hem raspbian

- Kopiera till SD-kort med:
    > diskutil list
    > diskutil unmountDisk /dev/diskN
    > sudo dd bs=8m if=raspbian.img of=/dev/diskN

- Starta rpi

- Uppdatera systemet (och expandera till fulla minneskortet)
    sudo raspi-config
    sudo rpi-update
    sudo apt-get update
    sudo apt-get dist-upgrade

- Ladda hem RetroPie-skriptet (via git) och kör
    sudo apt-get install -y git dialog
    cd
    git clone git://github.com/petrockblog/RetroPie-Setup.git
    cd RetroPie-Setup
    chmod +x retropie_setup.sh
    sudo ./retropie_setup.sh

- Fixa skärmen (TVn) med följande i /boot/config.txt (ställ mot 480p VGA)
    hdmi_group=1
    hdmi_mode=1
    overscan_scale=1

- Om ljudet är kasst, överklocka efter behag

- Fixa Xbox360-kontrollern
    sudo apt-get install xboxdrv

- Ladda in en instans (en kontroll) vid uppstart via /etc/rc.local:
    xboxdrv --trigger-as-button --wid 0 --led 2 --deadzone 4000 --silent &
    sleep 1

- Kontrollerna MÅSTE vara igång innan rpi:n startar

- Om den är buggig, testa trixa med USB-porten i /boot/cmdline.txt
    dwc_otg.speed=1

- Grundkonfa mot emulatorn
    cd ~/RetroPie/emulators/RetroArch/installdir/bin/
    ./retroarch-joyconfig -o player1.cfg -p 1 -j 0
    sudo cat player1.cfg >> ~/RetroPie/configs/all/retroarch.cfg


---

Uppdatera med spel (ROMs)
-------------------------

- Kopiera filerna till:
    /home/pi/RetroPie/roms

- Kan göras antingen via SAMBA shares eller över SSH med "scp"

---

Lägg till box art
-----------------

- Ladda hem ES-scraper
    cd RetroPie/supplementary
    git clone http://github.com/elpendor/ES-scraper

- Editera scraper.py ("PIL" fungerar inte, men "Image" gör)
    vim scraper.py
    Ändra "import os, imghdr, urllib, urllib2, sys, PIL, argparse, zlib"...
    till: "import os, imghdr, urllib, urllib2, sys, Image, argparse, zlib"...

- Kör skriptet via RetroPie-Setup ("SETUP" och sen "ES-Scraper")
    cd
    cd RetroPie-Setup
    sudo ./retropie_setup.sh