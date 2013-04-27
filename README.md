GNURadio prebuilt packages for Raspberry Pi on Archlinux armv6h
========================================================

Intro
-----
scateu@gmail.com

To install GNURadio on Raspberry Pi(archlinux armv6h), you just need to 

    yaourt -S gnuradio

but `gnuradio` needs `libuhd` which takes almost 5 hours to compile.

So I give my prebuilt packages.

                

How to compile
--------------
There are two methods. One is [cross-compile](http://archlinuxarm.org/developers/distcc-cross-compiling)(thanks to dheidler@gmail.com) , the other is compiling on rpi directly. Herein, we give the direct compiling details.

To compile all packages by yourself on your raspberry pi :

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

Tips:

You need to add arch "armv6h" to PKGBUILD file

or

    makepkg --asroot -Acs libuhd
    makepkg --asroot -Acs gnuradio

## Packages

### libuhd


Provide `libuhd-3.5.1-2-armv6h.pkg.tar.xz` for raspberry pi (archlinuxarm)

	pacman -U libuhd-3.5.1-2-armv6h.pkg.tar.xz 

Thanks to dheidler@gmail.com. He provided a newer version `libuhd-3.5.2-1-armv6h.pkg.tar.xz` for raspberry pi (archlinuxarm)

   

### GNURadio

Compiling for about 48hours.

        -- ######################################################
        -- # Gnuradio enabled components
        -- ######################################################
        --   * python-support
        --   * testing-support
        --   * volk
        --   * gruel
        --   * gnuradio-core
        --   * gr-fft
        --   * gr-filter
        --   * gr-atsc
        --   * gr-audio
        --   * gr-analog
        --   * gr-digital
        --   * gr-noaa
        --   * gr-pager
        --   * gr-trellis
        --   * gr-uhd
        --   * gr-utils
        --   * gr-vocoder
        --   * gr-fcd
        --   * gr-wavelet
        --   * gr-blocks
        --
        -- ######################################################
        -- # Gnuradio disabled components
        -- ######################################################
        --   * doxygen
        --   * sphinx
        --   * gnuradio-companion
        --   * gr-comedi
        --   * gr-qtgui
        --   * gr-shd
        --   * gr-video-sdl
        --   * gr-wxgui

when using the default PKGBUILD file, I came across an error as followings:

    [ 11%] Building C object gnuradio-core/src/lib/CMakeFiles/gnuradio-core.dir/filter/dotprod_fff_armv7_a.c.o
    {standard input}: Assembler messages:
    {standard input}:27: Error: selected FPU does not support instruction -- `vmov.f32 q8,#0.0'
    {standard input}:28: Error: selected FPU does not support instruction -- `vmov.f32 q9,#0.0'
    {standard input}:31: Error: selected processor does not support ARM mode `vld1.32 {d0,d1,d2,d3},[r0]!'
    {standard input}:32: Error: selected processor does not support ARM mode `vld1.32 {d4,d5,d6,d7},[r1]!'
    {standard input}:33: Error: selected FPU does not support instruction -- `vmla.f32 q8,q0,q2'
    {standard input}:34: Error: selected FPU does not support instruction -- `vmla.f32 q9,q1,q3'
    {standard input}:36: Error: selected FPU does not support instruction -- `vadd.f32 q8,q8,q9'
    {standard input}:37: Error: selected processor does not support ARMmode `vpadd.f32 d0,d16,d17'
    make[2]: *** [gnuradio-core/src/lib/CMakeFiles/gnuradio-core.dir/filter/dotprod_fff_armv7_a.c.o] Error 1
    make[1]: *** [gnuradio-core/src/lib/CMakeFiles/gnuradio-core.dir/all] Error 2
    make: *** [all] Error 2
    ==> ERROR: A failure occurred in build().
        Aborting...


so I change the `build()` function within PKGBUILD file like this:

    cmake -DPYTHON_EXECUTABLE=$(which python2)\
          -DPYTHON_INCLUDE_DIR=$(echo /usr/include/python2*) DPYTHON_LIBRARY=$(echo /usr/lib/libpython2.*.so) \
          -DCMAKE_INSTALL_PREFIX=/usr \
          -Dhave_mfpu_neon=0 \
          -DCMAKE_CXX_FLAGS:STRING="-march=armv6 -mfpu=vfp -mfloat-abi=hard" \
          -DCMAKE_C_FLAGS:STRING="-march=armv6 -mfpu=vfp -mfloat-abi=hard" \
          ../
               
seems work fine.

### rtl-sdr

have prebuilt package in pacman, just 

    pacman -S rtl-sdr

### gr-osmosdr-git
    
    yaourt -G gr-osmosdr-git

then change PKGBUILD file, add `armv6h` arch    

    arch=('i686' 'x86_64' 'armv6h')

then makepkg

    makepkg -As --asroot

### gr-air-modes

    pacman -S python2-numpy python2-scipy
    yaourt -G gr-air-modes-git

then change PKGBUILD file, add `armv6h` arch, and change depends `python` to `python2` so that you needn't to install python3

    arch=('i686' 'x86_64' 'armv6h')
    depends=('gnuradio' 'python2' 'gr-osmosdr-git' 'libuhd' 'sqlite' 'cmake')

comment out this line in file `/usr/lib/python2.7/site-packages/air_modes/__init__.py` in order not to import PyQt4 in poor RPi.
    
    from az_map import *

then enjoy :D

    modes_rx --gain=60 --output-all --rtlsdr --kml=xxx.kml
    

### their relationship map:

        libuhd
          |
        gnuradio      	rtl-sdr
          |                |
           --- gr-osmosdr--
                   |
                   |
               gr-air-modes
	


### gpsd

    pacman -S gbluez gpsd 

    chmod 666 /dev/ttyUSB0 # or gpsd will complain: gpsd:ERROR: /dev/ttyUSB0: device activation failed.

    gpsd -N -D5 /dev/ttyUSB0
    cgps
