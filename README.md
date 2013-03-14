GNURadio prebuilt packages for Raspberry Pi on Archlinux
========================================================

Intro
-----
scateu@gmail.com

To install GNURadio on Raspberry Pi(archlinuxarm), you just need to 

    yaourt -S gnuradio

but `gnuradio` needs `libuhd` which takes almost 5 hours to compile.

So I give my prebuilt packages.

                

How to compile
--------------
To compile all packages by yourself on your raspberry pi, 

### Resize your raspberry pi to full SD card
	
* fdisk  /dev/mmcblk0
	
 * p to see the current start of the main partition
 * d, 2 to delete the main partition
 * n p 2 to create a new primary partition, next you need to enter the start of the old main partition and then the size (enter for complete SD card)
 * w write the new partition table
	
* reboot
	
* `resize2fs /dev/mmcblk0p2`


        
### Make swap 
If you don't add swap partition for raspberry pi, errors like "unexpected }" will occur during compilation because of lacking of memory.

    swapoff
    swapoff -a
    sudo dd if=/dev/zero of=/myswap bs=1024 count=800k
    mkswap /myswap
    free -h
    swapon /myswap
    free -h
        
### Compile for long long time
    pacman -S file base-devel abs
    pacman -S yaourt
    yaourt -Syy
    yaourt -S gnuradio

or
    makepkg --asroot -Acs libuhd
    makepkg --asroot -Acs gnuradio

## Packages

### libuhd


Provide `libuhd-3.5.1-2-armv6h.pkg.tar.xz` for raspberry pi (archlinuxarm)

	pacman -U libuhd-3.5.1-2-armv6h.pkg.tar.xz 

### GNURadio




