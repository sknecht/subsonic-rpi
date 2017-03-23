# subsonic-rpi
HowTo — Subsonic with Java 8 on a Raspberry Pi (RPi) A step-by-step HowTo for a personal, headless and shared streaming music server.

(This is a clone of http://bit.ly/subsonic-RPi, an open GDoc — because someone said it'll be better to have it as a git.)

# Subsonic with Java 8 on a Raspberry Pi (RPi)

A step-by-step HowTo for a personal, headless and shared streaming music server.

Collaborative document — add, correct!

# Index

[[TOC]]

# Intro

## Motivation

As [Google Music](http://music.google.com) stops working with more that 100k tracks and my collection is larger, I gifted myself with a brandnew [RPi 3](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/) and an additional [HifiBerry-Amp](https://www.hifiberry.com/products/ampplus/). After fiddling around with several platforms, [Subsonic](http://www.subsonic.org/) came closest to what I liked: neat sorting and filtering, remotely accessing the music library and capable dealing with 100k+ tracks. At home, I am still using [Daphile](https://www.daphile.com/), a robust french application on a decomissioned, 8 year old ‘low-energy’ mini-PC with a bricky harddisk and connected to a HiFi with speakers through the PCs analog cinch. Beyond all, [Daphile](https://www.daphile.com/) is an audiophile’s heaven, dead simple to install yet has no obvious remote interface (or I didn’t find out).

What I have was a nicely sorted external 1TB USB HDD with tracks having correct ID3 tags and album art embedded. Using the utterly fantastic [mp3tag](http://www.mp3tag.de/en/) utility since long and even on a Mac as a Wine-app, over time the collection has been cleansed, normalized and sorted into a bunch of top-level genre collections per subdirectory. Turns out, this is a smart way of bringing a maintainable structure to Subsonic.

I enjoy music, all kinds, styles, genres independent of its technical quality. As personally I don’t hear much of a difference, audio is converted to 160kbit/s mp3s. Soe very old tracks are even lower and I couldn’t care less: my downstream equipment isn’t that high quality and neither are my ears. What I like is music of all styles — technical quality doesn’t matter too much for me: magic happens between my ears.

As a Linux newbie, [Subsonic](http://www.subsonic.org/) read like a match: having everything preconfigured, a lively development and Raspberry-friendly Debian images ready to use. I am fearless in Linux in applying someone elses’ recipes and mildly smart enough to adapt commands to my environment. If something goes wrong though, I’m lost. This mixup documents what I did, quoting others where credit is due and where I didn’t forget to copy the URL.

Use it — don’t blame me for anything. I will not be able to answer questions as I mostly don’t understand them … neither the answers. And: I did this using a Mac. Other OSes may slightly differ. Over the course of trying, branching and stumbling through new problems, I ran into a bunch of issues … and was able to solve them. 

So will you. 

**If you meet anything that needs correction: feel free and edit.**

## Hardware

The [RPi model 3](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/) was ordered at a german reseller [Max2Play](https://www.max2play.com/), together with a Class C 16GB SD with a Max2Play GUI to manage the RPi. However and for what suspected minor reason, I didn’t manage to run Subsonic in parallel to the umbrella image and decided to start from scratch, following several tutorials yet didn’t find a single step-by-step HowTo that I was able to follow and replicate. 

You may want to have attached a keyboard, (HDMI-connected) screen and a mouse. We’ll go headless if a [VNC-connection](#heading=h.oye9znah4n0h) is established and stable.

## Pointers to helpful other HowTos, linked in this document:

* [How to Turn a Raspberry Pi Into a Private Streaming Music Service](http://lifehacker.com/how-to-turn-a-raspberry-pi-into-a-private-streaming-mus-1583221462)

* [Subsonic on a Raspberry Pi](https://www.itsfullofstars.de/2015/05/subsonic-on-raspberry-pi/)

* [15 Useful Commands Every Raspberry Pi User Should Know](http://www.makeuseof.com/tag/15-useful-commands-every-raspberry-pi-user-should-know/)

* [A Raspberry Pi Subsonic Jukebox using Java 8](https://mj2p.co.uk/a-raspberry-pi-subsonic-jukebox-using-java-8/)

# Start: A blank RPi, a blank SD card

## Install the RPi OS, Jessie

Install the NOOBS image following [this tutorial](https://pimylifeup.com/noobs-raspberry-pi/). Nothing to add.

Do a SD-card backup if you feel like, [explained here](#heading=h.xr2b6yqx4bs). 

The Raspberry Pi default login for Raspbian is username pi with the password raspberry. 

Note that as you type your password nothing will be displayed. This is a security feature.

## Jessie installed. Configure RPi, install Java 8

### Install Oracle Java 8

sudo apt-get install oracle-java8-jdk

Check the Java version:

$ java -version

java version "1.8.0_65"
Java(TM) SE Runtime Environment (build 1.8.0_65-b17)
Java HotSpot(TM) Client VM (build 25.65-b01, mixed mode)

Looks good.

## Install Subsonic 6.0

(I followed [this tutorial](http://www.pihomeserver.fr/en/2013/05/11/raspberry-pi-home-server-raspbian-installer-un-serveur-subsonic/) and sneaked [there](https://www.itsfullofstars.de/2015/05/subsonic-on-raspberry-pi/).)

Determine the current stable Subsonic version on their [download page](http://www.subsonic.org/pages/download.jsp). Make sure your’re targeting the image and not some forwarding link. Copy the clean URL, then use wget and append this URL:

wget -O subsonic.deb http://subsonic.org/download/subsonic-6.0.deb

install the code to the RPi:

sudo dpkg -i subsonic.deb

see the installation progress:

sudo tail /var/subsonic/subsonic_sh.log

The installer configures your system to start Subsonic automatically when booting. After installing, open the Subsonic web page on [http://localhost:4040](http://localhost:4040)

For me and w/o a UI on the RPi, this translates to 192.168.178.37:4040

… and …

it works (yackie!):

![image alt text](image_0.png)

For first authentication, use the login (UID) and password (PWD) admin / admin

Good moment to have another snapshot and backup the SD, just in case some setup thingies go south. For me that happend once when trying to set new users and permissions. No idea what went wrong - reinstalled the saved image: all fine again.

## Edit config and modules

### Install the HiFi-Berry soundcard drivers for v8/Jessie

As my RPi has the hifiberry DAC+ installed, drivers need to be upgraded to Jessie. This is [explained here](https://www.hifiberry.com/upgrading-raspbian-to-jessie/).

The DAC+ and DAC+ Pro drivers were updated in kernel version 4.1.10+. However Jessie ships with the 4.1.7+ version of the kernel. To ensure you have the latest sound drivers you’ll need to update the kernel. This is best done using the rpi-update tool. From the command line run sudo rpi-update which will download and install the latest version of the kernel: 

For me, this read like

# firmware was successfully updated to af9cb14d5053f89857225bd18d1df59a089c171e

Whatever that means.

Reboot.

### Setup the correct Hifiberry Amp model

Ensure that the correct dtoverlay parameter, regarding the model of your DAC, is setup correctly in the /boot/config.txt file. For me, this is:

sudo nano /boot/config.txt

and add

dtoverlay=hifiberry-amp

or

dtoverlay=hifiberry-dacplus

CTRL+X saves the changes, confirm with Y

### Disable on-board RPi sound device

If you want the on-board raspberry sound devices to be disabled, from /etc/modules remove the line again with ... 

sudo nano /etc/modules

snd_bcm2835

*(In my setup, the above line wasn’t present, so nothing to change.)*

Reboot (not sure if neccessary).

Then again check if the HiFiBerry Amp+ positively is recognized and working:

aplay -l

**** List of PLAYBACK Hardware Devices ****

card 0: sndrpihifiberry [snd_rpi_hifiberry_amp], device 0: HifiBerry AMP HiFi tas5713-hifi-0 []

  Subdevices: 1/1

  Subdevice #0: subdevice #0

Looks fine. Proceed.

### Optional: Set the default sound card in Java

sudo nano /usr/bin/subsonic

add the following line before the -verbose:gc \ line

'-Djavax.sound.sampled.SourceDataLine=#ALSA [default]' \

tell the Pi where to send the sound (?=1 for speaker jack, 2 for HDMI, 0 for default)

sudo amixer -c 0 cset numid=3 ?

To reset to automatic just use

amixer cset numid=3 0

done. Restart the Pi

sudo reboot

Once you set up Subsonic with a media library you should be able to get sound directly from the RPi or via jukebox mode.

### Password-less login to the RPi

([source](http://blog.self.li/post/63281257339/raspberry-pi-part-1-basic-setup-without-cables))

Log out executing:

exit

… and copy your public ssh key into RPi with:

ssh-copy-id pi@[RPi-ip-address]

Now you should be able to ssh into RPi without password:

ssh pi@[RPi-ip-address]

Don’t have an SSH key? No problem. Follow [this guide from GitHub](https://help.github.com/articles/generating-an-ssh-key/) to create it.

## WiFi settings

Open terminal and edit /etc/wpa_supplicant/wpa_supplicant.conf on the SD card (not on your machine). ([source](http://blog.self.li/post/63281257339/raspberry-pi-part-1-basic-setup-without-cables))

cd /path/to/your/sd/card/

sudo nano etc/wpa_supplicant/wpa_supplicant.conf

and add the following to the bottom of the file:

network={

    ssid="your-network-ssid-name"

    psk="your-network-password"

}

### Wireless Configuration with WICD

See [Easy Wireless Configuration with wicd](https://blog.bartbania.com/raspberry_pi/easy-wireless-configuration-for-raspberry-pi/)

sudo apt-get update

sudo apt-get install wicd-curses

sudo wicd-curses

You’ll get a list of the wireless network found by the RPi. 

Use googles public DNS server 8.8.8.8 and 8.8.4.4 as well as any DNS servers your router is using.

### Installing WPA wireless tools 

SSH into the Raspberry Pi over ethernet. Install the WPA wireless tools: 

sudo apt-get install wpasupplicant wireless-tools

Edit the networking file to use the wpa-supplicant file for wireless configuration:

sudo nano /etc/network/interfaces

Ensure the wlan0 section uses the following lines: 

allow-hotplug wlan0

iface wlan0 inet manual

wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf

iface default inet dhcp

Optional: if static ip is desired replace "dhcp" above with "static" and add the following lines under it: 

address 10.0.0.200 (my static address outside of the dhcp)

netmask 255.255.255.0

network 10.0.0.0

broadcast 10.0.0.255

Edit the wpa-supplicant file 

sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

Ensure that it has the following lines

ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev

update_config=1

network={

ssid="YourSSID"

psk="password"

key_mgmt=WPA-PSK

}

Save the file, disconnect the ethernet cable, and reboot the Raspberry Pi. It should now connect to the wireless network.

Assuming your wireless card does not require drivers to be installed then it should be pretty simple to get WiFi-only working.

sudo nano /etc/network/interfaces

Change it to this:

auto lo

iface lo inet loopback

#Remove the wired network because it

#slows down boot if it's not connected.

#auto eth0

#iface eth0 inet dhcp

allow-hotplug wlan0

auto wlan0

iface wlan0 inet static

#(IP address of RPi)

address 192.168.1.20 

netmask 255.255.255.0

#(IP of router)

gateway 192.168.1.2 

wpa-ssid MyWirelessNetwork

wpa-psk MyPassword

## Optional: add an external WiFi Antenna

The RPi3 internal WLAN module is reasonable with delivering 43,7 Mbit/s (5,4 MBs). If however this isn’t good enough, you’re living in a bunker or a steel ship then an external WiFi-Module is to be considered as a small investment. 

The [Edimax EW-7612UAn Wireless-LAN USB-Adapter](https://www.amazon.de/gp/product/B007H5WXB0/ref=as_li_tl?ie=UTF8&camp=1638&creative=19454&creativeASIN=B007H5WXB0&linkCode=as2&tag=pow07-21&linkId=KGRNUG5FXIDQJBD6) (300Mbit/s) has [a good review](http://powerpi.de/raspberry-pi-3-wlan-geschwindigkeit-im-test/) and delivers 87,9 Mbit/s at 11,99 €. Didn’t try.

## Update the RPi

sudo apt-get update && sudo apt-get upgrade

This might take a while.

For my setup, it took whopping a 35+ minutes. *LibreOffice* was updated with lots of files - don’t understand why this was, probably as LibreOffice was part of the full RPi installation option. Don’t care, 16GB SD is plenty as my music payload will reside on an external HDD. Note to myself: uninstall LibreOffice, don’t need it anyway.*

## Set the correct locale

My locale has not been generated and LC_TYPE and LC_ALL are not set. Shouldn’t matter, the fallback is set to GB which is fine for me.

Generate the locale using ([source](https://www.thomas-krenn.com/de/wiki/Locales_unter_Ubuntu_konfigurieren))

sudo locale-gen de_DE.UTF-8

Solving the message "Warning: Setting locale failed." ([source](https://ubuntuforums.org/showthread.php?t=1720356))

$ locale

LANG=de_DE.UTF-8

LANGUAGE=

LC_CTYPE="de_DE.UTF-8"

LC_NUMERIC="de_DE.UTF-8"

LC_TIME="de_DE.UTF-8"

LC_COLLATE="de_DE.UTF-8"

LC_MONETARY="de_DE.UTF-8"

LC_MESSAGES="de_DE.UTF-8"

LC_PAPER="de_DE.UTF-8"

LC_NAME="de_DE.UTF-8"

LC_ADDRESS="de_DE.UTF-8"

LC_TELEPHONE="de_DE.UTF-8"

LC_MEASUREMENT="de_DE.UTF-8"

LC_IDENTIFICATION="de_DE.UTF-8"

LC_ALL=

The last line is empty, resulting in warning messages.

To fix this, add LC_ALL="de_DE.UTF-8" with

sudo nano /etc/environment

Reboot.

## Using Bluetooth on the RPi

*By default, it can't be used for audio. The Raspberry Pi is set to use either HDMI or 3.5mm output for audio. In order to get Bluetooth audio working, you will need to do **[a considerable amount of additional wor*k](https://www.raspberrypi.org/forums/viewtopic.php?t=68779)*.* ([source](https://www.cnet.com/how-to/how-to-setup-bluetooth-on-a-raspberry-pi-3/))

The [source](https://www.cnet.com/how-to/how-to-setup-bluetooth-on-a-raspberry-pi-3/) is fine, explains [how to pair a device through terminal](https://www.cnet.com/how-to/how-to-setup-bluetooth-on-a-raspberry-pi-3/).

Worked for me.

# Backup and Restore the SD

If you can afford, have several SDs available. Makes backups and occasional restores easier to handle.

## Backup SD status

([source](https://computers.tutsplus.com/articles/how-to-clone-raspberry-pi-sd-cards-using-the-command-line-in-os-x--mac-59911)) Backing up whats there, just in case to have a defined fallback if things went south in between. (I used that more than once, saves time and tears.) Until here, we have an OS, a VNC-Server running, the latest updates and a working WiFi-connection.

On the Mac, open a terminal, then diskutil list to identify the disk ID of the SD. 

My output is:

/dev/disk0 (internal, physical):

   #:                       TYPE NAME                    SIZE       IDENTIFIER

   0:      GUID_partition_scheme                        *251.0 GB   disk0

   1:                        EFI EFI                     209.7 MB   disk0s1

   2:          Apple_CoreStorage Macintosh HD            250.1 GB   disk0s2

   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3

/dev/disk1 (internal, virtual):

   #:                       TYPE NAME                    SIZE       IDENTIFIER

   0:                  Apple_HFS Macintosh HD           +249.8 GB   disk1

                                 Logical Volume on disk0s2

                                 AB4B4E89-6108-4814-9AD1-2E831166BB1A

                                 Unlocked Encrypted

/dev/disk2 (internal, physical):

   #:                       TYPE NAME                    SIZE       IDENTIFIER

   0:     FDisk_partition_scheme                        *16.1 GB    disk2

   1:             Windows_FAT_32 boot                    66.1 MB    disk2s1

   2:                      Linux                         16.0 GB    disk2s2

So obviously, disk2 is the SD to backup or clone to an image (img), the naked CLI command would be:

sudo dd if=/dev/disk2 of=~[pathto/backup-folder/filename.dmg]

I prefix the DMG with the date for later reference so if anything goes wrong, I have sort of an timestamp to wind back and start over.

sudo dd if=/dev/disk2 of=~/Downloads/Raspberry_Images/161227_RPi-OS_updated.dmg

Cloning the SD Card may take some time, no progress is shown though. I experienced 15min for a 16GB SD. A message shows up in Terminal if complete.

### Monitor dd progress with pipe viewer (pv), needs Homebrew

If you want to [monitor the progress](http://askubuntu.com/questions/215505/how-do-you-monitor-the-progress-of-dd) of the dd command, then pv is to be installed, [explained here](http://macappstore.org/pv/) for OSX.

Install [Homebrew](http://brew.sh/) to install pv then:

ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

And after, install pv to visualize the transfer proccess:

brew install pv

From [the package description](https://www.ivarch.com/programs/pv.shtml): *pv** - Pipe Viewer - is a terminal-based tool for monitoring the progress of data through a pipeline. It can be inserted into any normal pipeline between two processes to give a visual indication of how quickly data is passing through, how long it has taken, how near to completion it is, and an estimate of how long it will be until completion.*

**Example**

dd if=/dev/urandom | pv | dd of=/dev/null

**Output**

1,74MB 0:00:09 [ 198kB/s] [ <=> ]

You could specify the approximate size with the --size if you want a time estimation.

Command with pv:

sudo dd if=/dev/sdb | pv -s 2G | dd of=DriveCopy1.dd bs=4096

For my setup, the command then reads:

sudo dd if=/dev/disk3 | pv -s 16G -e -t | dd of=~/Downloads/Raspberry_Images/170113_RPi-sub-indexed.dmg bs=4096

No idea what the bs=4096 does — said to be faster that way.

Plain text (note to myself): "as a superuser, run an image from the device disk 3 and while doing so estimate the duration of the process of this 16GB operation while copying the data to a DMG-file named YYMMDD_some-name.dmg"

### Converting IMG <> DMG

Converting the IMG to DMG on OSX is explained in detail [here](https://www.techwalla.com/articles/how-to-convert-img-to-dmg-for-a-mac). If done, stick the SD back in the RPi and reboot.

The other way, converting DMG to IMG is:

hdiutil convert disk.dmg -format RdWr -o disk.img

… where disk.dmg is the path to your image (input) and disk.img the output image

If the CLI throws an error hdiutil: convert failed - image/device is too large, then I used [DropDMG](https://c-command.com/dropdmg/), a UI-tool that does the job. Payware, 23,99 $ — no idea why this error occurs though, didn’t investigate.

After making a backup of a SD Card, and trying to restore it using 'Disk Utility' you may have some issues during the process like:

Error: Could not validate source - Invalid argument

To solve this, there is this 3 step process that you can perform in the terminal. If you do not want to run this steps, there is [a script that writes back a dmg to SD](https://github.com/magamig/dmgtosd): [dmgtosd.sh](https://raw.githubusercontent.com/magamig/dmgtosd/master/dmgtosd.sh) To be able to run this script, Homebrew needs to be installed already.

If so, run [dmgtosd.sh](https://raw.githubusercontent.com/magamig/dmgtosd/master/dmgtosd.sh) as sudo (follow the link before for the script itself).

**Somewhere I learned, a ****dmg**** can just be renamed into an ****img**** which is way easier of course.**

## Restore/Write back an IMG to SD: Etcher or CLI

Writing an IMG to SD can be done with terminal. 

sudo dd if=path_of_your_image.img of=/dev/rdiskn bs=1m

**Or just use →****[Etche**r](http://etcher.io)**, a foolproof and free multi-platform tool with a graphical UI, available for many OSes, recommended.**

# Setting up an external, USB-HDD

Now for the structural part: you want the external HDD be detected and mounted after reboots, glitches whatever.

## HDD: Size of Partitions

The RPi can handle partitions up to 2TB. Larger disks have to be partitioned accordingly. No hints here, you’ll find that easily with your trusted search engine.

## Add HDD disk format drivers

To end the disk format struggles, now drivers are to be added to the raw Jessie install so that commonly used disk formats are read by the RPi, nicely [explained here (German only)](https://jankarres.de/2013/01/raspberry-pi-usb-stick-und-usb-festplatte-einbinden/):

sudo apt-get -y install ntfs-3g hfsutils hfsprogs exfat-fuse

sudo apt-get install exfat-fuse exfat-utils

For brevity, only the exFAT and FAT32 option is elaborated here, other filesystems need to be set to whatever collateral conditions.

<table>
  <tr>
    <td>FAT32</td>
    <td>sudo mount -t vfat -o utf8,uid=subsonic,gid=subsonic,noatime /dev/sda2 /media/knemukke2</td>
  </tr>
  <tr>
    <td>exFAT</td>
    <td>sudo mount -t exfat -o utf8,uid=subsonic,gid=subsonic,noatime /dev/sda2 /media/knemukke2</td>
  </tr>
  <tr>
    <td>NTFS</td>
    <td>sudo mount -t ntfs-3g -o utf8,uid=pi,gid=pi,noatime /dev/sda /media/usbstick</td>
  </tr>
  <tr>
    <td>HFS+</td>
    <td>sudo mount -t hfsplus -o utf8,uid=pi,gid=pi,noatime /dev/sda /media/usbstick</td>
  </tr>
  <tr>
    <td>ext4</td>
    <td>sudo mount -t ext4 -o defaults /dev/sda /media/usbstick</td>
  </tr>
</table>


Mount parameters after -o (for option)

* nls=utf8: UTF-8 is used and can contain Umlauts (äöüß) for German

* uid=subsonic,gid=subsonic: the drive is attached to user (uid) and group (gpi) ‘pi’

* noatime: inode access time (last read) is not  updated, unneccessary write operations are avoided

* defaults: the device takes rw, suid, dev, exec, auto, nouser and async (use the search engine you trust to decipher these)

<table>
  <tr>
    <td>FAT32</td>
    <td>UUID=231C-1C01 /media/KNEMUKKE2 vfat nls=utf8,uid=subsonic,gid=subsonic,noatime 0</td>
  </tr>
  <tr>
    <td>NTFS</td>
    <td>UUID=3241-40CE /media/usbstick/ ntfs-3g utf8,uid=pi,gid=pi,noatime 0</td>
  </tr>
  <tr>
    <td>HFS+</td>
    <td>UUID=3241-40CE /media/usbstick/ hfsplus utf8,uid=pi,gid=pi,noatime 0</td>
  </tr>
  <tr>
    <td>exFAT</td>
    <td>UUID=3241-40CE /media/usbstick/ exfat utf8,uid=pi,gid=pi,noatime 0</td>
  </tr>
  <tr>
    <td>ext4</td>
    <td>UUID=3241-40CE /media/usbstick/ ext4 defaults 0</td>
  </tr>
</table>


Done. Now the external USB-HDD will be mounted on each reboot.

## Better: automount connected USB HDDs with usbmount

As a complete dummy, [this hint](https://krausens-online.de/debianraspbian-usb-automatisch-mounten/) came in just-in-time: 

sudo aptitude install usbmount

This generates a directory /media from root containing further directories „usb0" to „usb7“ plus a symlink „usb“ that references to „usb0“. If an external USB-HDD is attached to the RPi, it is mounted automagically.

If you’re root, mounted USB-devices should be good to go and useable. If you are another user, like UID Pi, then the USB-HDD can only be read, not written to. All other users only can read: rwxr-xr-x

In order to have r/w access for other users, another step is to be taken:

sudo nano /etc/usbmount/usbmount.conf

Search for ...

# For example, "-fstype=vfat,gid=floppy,dmask=0007,fmask=0117" would add

# the options "gid=floppy,dmask=0007,fmask=0117" when a vfat filesystem

# is mounted.

FS_MOUNTOPTIONS=""

and change to

FS_MOUNTOPTIONS="-fstype=vfat,gid=users,dmask=0007,fmask=0117"

Reboot.

## Define a mount point for your external USB HDD

Then create a mount point so that we can easily access the files on the external hard drive once we’ve told the system where to find it. This is a folder in /media a symbolic link (?) to the USB-drive. ([source](https://mj2p.co.uk/add-an-external-hard-drive-to-a-subsonic-raspberry-pi-server/)) The folder name can be freely set, shouldn’t contain special characters. 

I name it knemukke2 alike the name of my external 1TB USB HDD.

pi@hifiberry:cd /media

pi@hifiberry:sudo mkdir knemukke2

pi@hifiberry:pwd

pi@hifiberry:/media/knemukke2

Then change ownership of the mount point to the user that runs the subsonic process:

sudo chown subsonic /media/knemukke2

sudo chgrp subsonic /media/knemukke2

## Identify the HDD UID to add it to fstab

Now edit the fstab file which lists the devices the RPi can mount. It is read at boot so by adding to it we can make the external drive accessible every time the Pi boots.

To do so, we need to know the unique ID of the hard drive:

sudo blkid

This will give an output similar to this:

/dev/mmcblk0p1: SEC_TYPE="msdos" LABEL="boot" UUID="7F22-B6C1" TYPE="vfat" PARTUUID="8936756d-01"

/dev/mmcblk0p2: UUID="402bfe3d-37db-48a7-a515-31edccf953df" TYPE="ext4" PARTUUID="8936756d-02"

/dev/mmcblk0: PTUUID="8936756d" PTTYPE="dos"

Ooops - the external USB HDD knemukke2 doesn’t show up. Why is that?

It’s not *mounted* obviously. *(Hours later … a faulty USB cable was the &$&% culprit)*

To see all mount points:

sudo blkid -o list -w /dev/null

For me this looks like …

device      fs_type label    mount point     UUID

----------------------------------------------------------------------------------

/dev/mmcblk0p1

            vfat    boot     /boot           7F22-B6C1

/dev/mmcblk0p2

            ext4             /               402bfe3d-37db-48a7-a515-31edccf953df

/dev/mmcblk0

                             (in use)

/dev/sda1   vfat    EFI      (not mounted)   67E3-17ED

/dev/sda2   vfat    KNEMUKKE2 /media/knemukke2 231C-1C01

Here we go - /dev/sda2 is my HDD, I now know that the mount point /media/knemukke2 is linked to /dev/sda2 and the HDD-UID is 231C-1C01.

## Automatically bind and use the external USB-HDD on reboot

The automatic binding makes sense as the external HDD is where all the music is to be, adressable with Subsonic.

Do so with adding an entry in fstab — we need the UUID that previously has been read out with the blkid command. Again, this depends on the filesystem you use. One of the following lines has to be added to the end of /etc/fstab - depending on the disk format you deploy.

Again, use nano as an editor, save with STRG + X, Y and Enter to confirm overwriting the existing file with new content.

sudo nano -w /etc/fstab

I added this line - customized this to your HDDs UUID and settings.

Important: editing fstab, use Tabs instead of blanks!

Then I added:

UUID=231C-1C01 /media/usb/knemukke2 vfat utf8,uid=subsonic,gid=subsonic 0       0

Mount/Unmount the external USB-HDD to test if it’s working OK:

sudo mount /media/usb/knemukke2

sudo unmount /media/usb/knemukke2

## Troubleshooting: external USB HDD consumes too much energy

You certainly know that itchy sound as an external HDD starts, spins, hick-ups and powers down repeatedly? At least in peaks, conventional external HDDs consume more energy than the RPi has available through USB. That’s ugly.

RPi [official power requirements](https://www.raspberrypi.org/help/faqs/#topPower) says:

*The correct Power Supply for the Raspberry Pi 3 is ***_5V 2.5 Amps_*** if you continue using what you currently have you may burn out your Raspberry Pi 3 or HD or even both! *

*Also the Raspberry Pi 3 by it self does not draw much 550 mAh at starting then drops down to 300 to 350 mAh at idle but you need to remember also if you turn on the WiFi, Bluetooth, USB Thumb Drive and add maybe a USB HD Pi Drive to the mix your drawing more energy there for the 5V 2.5 Amp comes in.* ([quote](https://www.raspberrypi.org/blog/meet-314gb-pidrive/#comment-1255940))

Setting the USB bus to high-current mode with ...

Set max_usb_current=1 in config.txt. 

### Alternative modes of powering the external HDD 

1. Power the HDD through an externally powered USB-Hub and a Y-shaped cable that draws energy from elsewhere

2. get a less energy-hungry HDD.

Exactly this was the solution I was searching for: one of those forked cables. Had one, installed it: the shorter USB-end goes into the powered hub to draw energy, the longer one plugs into the RPi. Works.

Disadvantage: the HDD is ON even if the RPi is shutdown as the external USB-Hub still delivers power. So just another thing to remember: powering off the Pi + powering off the HDD.

# Add a user for subsonic and add it to the audio group

You don’t want subsonic to be run as root as this might open all doors for those unfriendly hackers out there. So give it its own user:

sudo adduser subsonic

sudo adduser subsonic audio

## Install Watchdog to reboot the RPi if it’s getting unresponsive

([source](http://blog.self.li/post/63281257339/raspberry-pi-part-1-basic-setup-without-cables))

sudo apt-get install watchdog

sudo modprobe bcm2708_wdog

sudo nano /etc/modules

And at the bottom add:

bcm2708_wdog

Now let’s add watchdog to startup applications:

sudo update-rc.d watchdog defaults

… and edit its config:

sudo nano /etc/watchdog.conf

#uncomment the following:

max-load-1

watchdog-device

Start watchdog with:

sudo service watchdog start

# Connect to the RPi through VNC

[Download](https://www.realvnc.com/download/viewer/) and run VNC Viewer.

In my case, the IP of the RPi is 192.168.178.37:5900 — 5900 being the port it listens to.

![image alt text](image_1.jpg)

OK - connect … works like a warm knife through frozen butter: I can remotely interact with the RPi GUI now. That’s much better.

Good time for a →[SD backup](#heading=h.j82j0jusrdta).

# Magic: systemd tweaking

Here comes the vodoo. I don’t understand exactly what it was — yet without the help of a friendly Linux-Wizz from Dumbledore HQ, I’d be completely stuck there. Thank you, Florian! 

Trying to explain, then telling what to do.

Subsonic comes as a Debian package to be installed including a script to start subsonic. That script starts subsonic into the background using a parameter "&". As the script is automatically loaded on boot through systemd and systemd attempts to monitor the status, problems come up. If subsonic switches into an background app, systemd understands this as not being started correctly and signals subsonic as "active, not running" which translates to "autostart entry is active but app is not running".

The golden bullet came from [DerArne](http://linux-club.de/forum/memberlist.php?mode=viewprofile&u=37017&sid=776734fa96f0c57bbd9e42628457138e) explaining how to start subsonic with systemd ([German source](http://linux-club.de/forum/viewtopic.php?t=116708)) for OpenSuSE yet applicable for Debian.

Generate a file /etc/systemd/system/subsonic.service with this content:

[Unit]

Description=Subsonic Sound-Server

[Service]

User=subsonic

WorkingDirectory=/opt/subsonic

ExecStart=/opt/subsonic/subsonic.sh 

Restart=on-abort

[Install]

WantedBy=multi-user.target

Quote: 

(…) *the core problem with this script is that the Java process is started in its own thread. After starting, the script terminates itself and the Java process runs as background daemon process. What’s fine for a trial on the desktop conflicts the logic of *systemd* that assumes the started script has died already. From a *systemd* perspective, the script went wrong and everything is terminated — including the Java process.*

*To solve the issue, delete the ‘&’ from the java process start. Now the script waits until the Java process terminates … and *systemd* immediatey restarts it. Furthermore, the startscript also handles a PID and start parameters which we don`t need under *systemd.

So the final script reads like this:

#!/bin/sh

############################################################################

# Shell script for starting Subsonic.  See http://subsonic.org.

#

# Author: Sindre Mehus

############################################################################

SUBSONIC_HOME=/opt/subsonic

# your correct RPi IPv4 here

SUBSONIC_HOST=192.168.7.20

SUBSONIC_PORT=4040

SUBSONIC_HTTPS_PORT=0

SUBSONIC_CONTEXT_PATH=/

SUBSONIC_MAX_MEMORY=150

# your correct default media folders here

SUBSONIC_DEFAULT_MUSIC_FOLDER=/data/Musik

SUBSONIC_DEFAULT_PODCAST_FOLDER=/data/Musik

SUBSONIC_DEFAULT_PLAYLIST_FOLDER=/data/Musik

quiet=0

# Use JAVA_HOME if set, otherwise assume java is in the path.

JAVA=java

if [ -e "${JAVA_HOME}" ]

    then

    JAVA=${JAVA_HOME}/bin/java

fi

# Create Subsonic home directory.

mkdir -p ${SUBSONIC_HOME}

LOG=${SUBSONIC_HOME}/subsonic_sh.log

rm -f ${LOG}

cd $(dirname $0)

if [ -L $0 ] && ([ -e /bin/readlink ] || [ -e /usr/bin/readlink ]); then

    cd $(dirname $(readlink $0))

fi

${JAVA} -Xmx${SUBSONIC_MAX_MEMORY}m \

  -Dsubsonic.home=${SUBSONIC_HOME} \

  -Dsubsonic.host=${SUBSONIC_HOST} \

  -Dsubsonic.port=${SUBSONIC_PORT} \

  -Dsubsonic.httpsPort=${SUBSONIC_HTTPS_PORT} \

  -Dsubsonic.contextPath=${SUBSONIC_CONTEXT_PATH} \

  -Dsubsonic.defaultMusicFolder=${SUBSONIC_DEFAULT_MUSIC_FOLDER} \

  -Dsubsonic.defaultPodcastFolder=${SUBSONIC_DEFAULT_PODCAST_FOLDER} \

  -Dsubsonic.defaultPlaylistFolder=${SUBSONIC_DEFAULT_PLAYLIST_FOLDER} \

  -Djava.awt.headless=true \

  -verbose:gc \

  -jar subsonic-booter-jar-with-dependencies.jar > ${LOG} 2>&1

If that is changed accordingly, then subsonic can be registered with systemd:

systemctl enable subsonic.service

… then started:

systemctl start subsonic.service

To check if everything went as planned, use ...

systemctl | grep subsonic

… to result in:

subsonic.service          loaded active running       Subsonic Sound-Server

All fine.

# Configure Subsonic for remote media streaming

## Open port 4040 on your router

On your router: set a **static IP-address and open port 4040 for TCP/IP**

Beware: that is somehow easy for IPv4 and still beyond my understanding for IPv6. An indication for IPv6 on your router is a line saying "DS-Lite-Tunnel" ("Dual Stack Lite").

There seem to be IPv6 portmapper ([German link](https://jankarres.de/2015/08/raspberry-pi-dyndns-trotz-ds-lite-durch-ipv6-portmapper/)) yet no further insight here.

## Give your Subsonic a subdomain on subsonic.org

Costs 1$/month after 30 free days. Fully worth it, me thinks.

The alternative is to setup your own DynDNS as the DSL provider will reset your IP from time to time. DynDNS covers this as your router’s changing external address will be signaled tp the dynamic DNS provider and soehow aliased so that a persistent, readable machine name will always point to your RPi behind your router. There are free dynamic DNS services. Follow this [HowTo](https://pavelfatin.com/access-your-raspberry-pi-from-anywhere/) if you’re up for this.

For me, the success was instant and rewarding:

Status: knemukke.subsonic.org responded successfully. 

## Use HTTPS for the Hifiberry

add --https-port=4443 (or any other port) to SUBSONIC_ARGS in /etc/default/subsonic (Ubuntu/Debian) 

# End: A headless Subsonic media server on a RPi3.

# Appendix: Subsonic configuration hints

### Logfiles and commands

log file is available at /var/subsonic/subsonic_sh.log

sudo nano /var/subsonic/subsonic_sh.log

### Subsonic Start/Stop/Restart/Status

sudo /etc/init.d/subsonic {start|stop|status|restart|force-reload}

### RPi shutdown

sudo shutdown -h 0

### Make the user subsonic own subsonic processes

Subsonic stores its data in default folders. By default, for Debian it is /var/subsonic. 

Because subsonic was already started, this folder is created and filled with content, using the default subsonic user: root (yep, BAD). ([source](https://www.itsfullofstars.de/2015/05/subsonic-on-raspberry-pi/))

As the subsonic-user is set already (above), now subsonic is to be set as to use that user and run under that user id and not as root. The user information is stored in the default subsonic configuration file: /etc/default/subsonic

sudo nano /etc/default/subsonic

The last line must be changed to: SUBSONIC_USER=subsonic

### Change port, Java memory settings and other parameters

To change the port number, Java memory settings or other startup parameters, edit the SUBSONIC_ARGS variable in /etc/default/subsonic

sudo nano /etc/default/subsonic

SUBSONIC_ARGS="--port=4040 -- https-port=443 --init-memory=256 max-memory=512 --context-path=/subsonic"

I didn’t change anything and neither did set the initial Java memory allocation. Runs nicely.

### Setting Permissions for user subsonic

Make user subsonic owner of /var/subsonic

sudo chown subsonic:subsonic /var/subsonic –Rv 

### FTP to upload files to the USB-HDD attached

This section is incomplete. 

In order to remotely transfer data to the USB-HDD attached to the RPi: FTP has best throughput performance. HowTo from →[here](https://pimylifeup.com/raspberry-pi-ftp/) 

sudo apt-get update

sudo apt-get install vsftpd

Now open up the config file by entering 

sudo nano /etc/vsftpd.conf

In here add or uncomment (remove the #) for the following settings.

anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
chroot_local_user=YES
user_sub_token=$USER
local_root=/home/$USER/ftp

Once you are done press ctrl+x and then y to save and exit.

Now we need to create the FTP directory so we can transfer files. The root directory is not allowed to have write permissions so we will need a sub folder called files. If you try to copy to FTP it won’t work but FTP/files will. Replace <user> with the relevant user, for example

mkdir /home/<user>/FTP
mkdir /home/<user>/FTP/files
chmod a-w /home/<user>/ftp

Now restart the service by entering the following command

sudo service vsftpd restart

You should now be able to connect over plain FTP (Port 21) from within your WiFi. For external connectivity, Port 21 has to be opened on your home router, ideally with another port number to provide minimal security from hacker kiddies.

# Tooling

This is a random collection of Linux or RPi tools.

### PiBakery: visually arrange basic RPi settings

[Download](http://www.pibakery.org/docs/install-mac.html), install, run PiBakery. 

Steps to follow are [nicely explained](http://www.pibakery.org/docs/create.html).

This is good for setting basics without prior experience or knowledge. Just arrabge the commands you want and proceed.

Insert SD, insert power. 

Wait a minute to boot.

### Useful CLI (command line) commands for the absolute Dummy

([source](http://blog.helmutkarger.de/raspberry-media-center-teil-23-ssh-und-einige-nuetzliche-linux-befehle/))

<table>
  <tr>
    <td>ssh pi@192.168.178.37</td>
    <td>Connect with SSH (my setup, change IP for yours)
Open a terminal window and connect: (source) iTerm on OSX is a nice alternative to the built-in terminal app.
Advanced: Secure SSH access, public key pair (source)</td>
  </tr>
  <tr>
    <td>UID pi
PWD raspberry</td>
    <td>Defaul RPi login</td>
  </tr>
  <tr>
    <td>sudo</td>
    <td>Use sudo for commands that need more privileges than your current user.</td>
  </tr>
  <tr>
    <td>sudo ifconfig</td>
    <td>Show network interfaces</td>
  </tr>
  <tr>
    <td>lsusb</td>
    <td>Show connected USB devices</td>
  </tr>
  <tr>
    <td>sudo reboot</td>
    <td>Reboot via SSH</td>
  </tr>
  <tr>
    <td>sudo shutdown</td>
    <td>Shutdown via SSH</td>
  </tr>
  <tr>
    <td>passwd</td>
    <td>Change password</td>
  </tr>
  <tr>
    <td>uname -a</td>
    <td>Which version is running now?
uname -a
Linux hifiberry 4.4.21-v7+ #911 SMP Thu Sep 15 14:22:38 BST 2016 armv7l GNU/Linux</td>
  </tr>
  <tr>
    <td>cat /etc/os-release</td>
    <td>PRETTY_NAME="Raspbian GNU/Linux 8 (jessie)"
NAME="Raspbian GNU/Linux"
VERSION_ID="8"
VERSION="8 (jessie)"
ID=raspbian
ID_LIKE=debian
HOME_URL="http://www.raspbian.org/"
SUPPORT_URL="http://www.raspbian.org/RaspbianForums"
BUG_REPORT_URL="http://www.raspbian.org/RaspbianBugs"</td>
  </tr>
  <tr>
    <td>sudo tail -f /var/subsonic/subsonic_sh.log</td>
    <td>view subsonic logfiles</td>
  </tr>
  <tr>
    <td>journalctl -p err -b</td>
    <td>only show system log entries with an error level

</td>
  </tr>
</table>


### Install Midnight Commander

… a visual, tabbed file manager alike the good old Norton Commancer — when amber monitors were sexier than black/green ones..

$ sudo apt-get md install

Start Midnight Commander from the console: 

$ sudo mc

### Find your RPI on the LAN

[PIfinder](http://ivanx.com/raspberrypi/)

*If you just booted up your Raspberry Pi, but don't have it connected to a screen and keyboard, you probably want to log in from a Terminal window, but don't know its IP address. Pi Finder figures it out for you, and makes it easy to connect.*

*Pi Finder also shows you the MAC (Ethernet) address of your Raspberry Pi, so you can add it to your router as a DHCP reservation, meaning your Raspberry Pi will get the same IP address every time and you won't need to run Pi Finder again.*

[download Pi Finder 1.1.1](http://ivanx.com/raspberrypi/) [for OS X 10.5 Leopard through 10.10 Yosemite]

### Reset A Forgotten Raspberry Pi Password

[http://www.raspberrypi-spy.co.uk/2014/08/how-to-reset-a-forgotten-raspberry-pi-password/](http://www.raspberrypi-spy.co.uk/2014/08/how-to-reset-a-forgotten-raspberry-pi-password/)

### BerryBoot v2.0 - bootloader / universal operating system installer

Download link Berryboot for the quad-core Raspberry Pi 2 and Pi 3 only (36 MB): [berryboot-20170208-pi2-pi3.zip](http://downloads.sourceforge.net/project/berryboot/berryboot-20170208-pi2-pi3.zip)

# ToDos (private)

Any hints welcome.

1. Make Subsonic’s incoming dir writable for any Subsonic user so that uploaded files are at one place.

2. Change the default RPi password

3. Make Bluetooth working so an external BT Speaker can take the signal

4. Backup SD automatically: [http://forum.subsonic.org/forum/viewtopic.php?f=6&t=10900](http://forum.subsonic.org/forum/viewtopic.php?f=6&t=10900) and https://raspberry.tips/raspberrypi-einsteiger/raspberry-pi-datensicherung-erstellen/

5. private: make the subsoic DLNA-server interact with the local Daphile instance

6. private: *uninstall LibreOffice, don’t need it anyway.*

7. Enjoy.


