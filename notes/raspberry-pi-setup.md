# Raspberry Pi Setup
there are tons of tutorials about how to setup a Raspberry Pi, see [resources](#resources). for sake of centralized and exhaustive documentation, we are including how we set things up here in the hope that it will be helpful for passersby.

## Download OS
since the pi doesn't have an operating system on it, we need to download one and the flash it to an SD card which we will use as our hard drive. we are going to use the latest version of *Raspbian* (a variant on the linux *Debian* distribution) called *Stretch* (as of 4.8.2018). in particular we are going to use the *lite* version of this OS since we don't need a desktop environment; we will only be accessing the pi using the command line!

you can download the zip of the OS image by opening your terminal and typing,

``` shell
$ wget https://downloads.raspberrypi.org/raspian_lite_latest
```

this will download the zip file (`~350MB`) into the current directory you are in; it might take a few minutes depending on your network connectivity. Alternatively, you can visit the [Official Raspberry Pi Downloads Page](https://www.raspberrypi.org/downloads/raspbian/) and click the download button.

## Flash OS Image -> SD Card
now that we have the operating system downloaded, we want to put that operating system on an SD Card which we can then plug into the pi which will use it as its primary hard drive (the resulting disk will contain a boot and root partition).

we had a 4GB SanDisk SD card (not a micro-SD card, but one of those old big ones since this older model of the pi takes a big one). the *Raspbian Stretch-lite* image should be `~1.8GB` so it should fit onto this SD card. You can totally get a bigger SD card if you want, we just had this lying around.

#### (1) unzip
first, we need to unzip the OS image we just downloaded,

``` shell
$ unzip raspbian-stretch-lite.zip
```

if you list the files in your directory you should now see `raspbian-stretch-lite.img`! and just to make sure that it will fit on the SD card that you intend to use with your pi, you can check the size of the image.

``` shell
$ ls -lah raspbian-stretch-lite.img
 -rw-r--r-- 1 c users 1.8G Mar 13 18:53 raspbian-stretch-lite.img
```
great, our image is `1.8GB` (as we assumed earlier).

#### (2) locate SD card device
this step will depend on your computer's hardware and OS-- i am using Archlinux on a machine which doesn't have an SD card reader on it. So i will be using an external USB SD card reader/writer. if you have a built-in SD card reader or your USB devices are auto-mounted, you can refer to [these instructions](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).

plug in your SD card into your SD card reader and look at the newly connected devices,

``` shell
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdd           8:48   1   3.7G  0 disk
nvme0n1     259:0    0 953.9G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part /boot
├─nvme0n1p2 259:2    0    20G  0 part /
└─nvme0n1p3 259:3    0 429.5G  0 part /home
```
your results will differ from mine, but i can see that there is a connected device of type `disk` named `sdd` which is `3.7GB` this looks like our SD card!

let's look in our devices directory to check to see if that device is in there,

``` shell
$ ls /dev
```
you should see it in the results :)

#### (3) flash image to SD card
i will be using a built-in linux command called `dd` (to convert and copy a file) to flash the OS image onto the SD card. user beware! `dd` can overwrite partitions on your computer if you do not use it correctly! for less dangerous options, one could consider using [Etcher](https://etcher.io/) as recommended by the official Raspberry Pi docs. i like `dd` because it is a straight-forward command and does exactly what we want without having to install anything extra. plus i always find it really refreshing when i understand things at a lower level of abstraction. *aaaahhhhhhhh*- a refreshing sip of `dd`.


![alt-text](https://media.giphy.com/media/a0q8vE3WKTIzK/giphy.gif)

to flash the OS image to the SD card we perform the following command,

``` shell
$ sudo dd bs=4M if=raspbian-stretch-lite.img of=/dev/sdd conv=fsync
```

* `bs` is the block size. (4m is the natural block size in SD storage, so specifying this will make the transfer go faster.).
* `if` is the input file that we actually want to flash to our SD storage-- in our case it is the Raspbian OS image file.
* `of` is the output file which corresponds to the device we want to be flashing our OS image to-- in our case this is `/dev/sdd` (note though, that your SD storage device will probably have a different name so you need to use that).
* `conv=fsync` basically tells the `dd` command to actually persist the data onto the disk after the operation is complete (this is important because sometimes for optimization reasons (?) the data is only persisted in memory and not on disk???).

this command will take a few minutes to complete, so maybe you can get a cup of coffee and chat with your pals.

#### (4) mount our new raspbian boot & root partitions
after we have flashed the OS to our SD storage, we should now have a boot and root partition on our SD storage disk. we can verify this by again running `lsblk` and see some nested partitions,

``` shell
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdd           8:48   1   3.7G  0 disk
├─sdd1        8:49   1  41.8M  0 part
└─sdd2        8:50   1   1.7G  0 part
nvme0n1     259:0    0 953.9G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part /boot
├─nvme0n1p2 259:2    0    20G  0 part /
└─nvme0n1p3 259:3    0 429.5G  0 part /home
```
cool, we now have `sdd1` (boot partition) and `sdd2` (root partition) as partitions of our `sdd` disk.

lets verify that everything looks good by mounting each partition and checking them visually. mount this partitions onto our computer's filesystem using `mount`. i typically mount disks in the `/mnt/` directory. lets mount and look at the boot partition, it should look like this,

``` shell
$ sudo mount /dev/sdd1 /mnt/usb1
$ ls /mnt/usb1
bcm2708-rpi-0-w.dtb     bcm2710-rpi-3-b.dtb       config.txt     fixup_x.dat       LICENSE.oracle  start_x.elf
bcm2708-rpi-b.dtb       bcm2710-rpi-3-b-plus.dtb  COPYING.linux  issue.txt         overlays
bcm2708-rpi-b-plus.dtb  bcm2710-rpi-cm3.dtb       fixup_cd.dat   kernel7.img       start_cd.elf
bcm2708-rpi-cm.dtb      bootcode.bin              fixup.dat      kernel.img        start_db.elf
bcm2709-rpi-2-b.dtb     cmdline.txt               fixup_db.dat   LICENCE.broadcom  start.elf
$ sudo umount /mnt/usb1
```

lets mount and look at the root partition, it should look like this,

``` shell
$ sudo mount /dev/sdd2 /mnt/usb1
$ ls /mnt/usb1
bin  boot  dev  etc  home  lib  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
$ sudo umount /mnt/usb1
```


## Resources
[Official Raspbian Installation Guide](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)
