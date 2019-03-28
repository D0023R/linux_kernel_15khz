# This repository contains the Linux kernel 15kHz video patches

The provided kernel patches enable the 15kHz video support with additional features for Linux.

## CONTENT:

- 01_ati_9200_pllfix.diff (only required to support ATI 9200 card model)
- 02_arcadevga_3000.diff (only required to support ARCADEVGA 3000 card model)
- 03_linux_15khz.diff (main patch for 15 kHz support)
- 04_linux_15khz_scanoutpos.diff (optional, enable scanline position reporting to use space, will be required to support future groovymame synchronisation library)
- 05_linux_15khz_interlaced_mode_fix.diff (fix the vertical blank interrupt reporting to make interlaced resolution workings with groovymame)

## KERNEL COMPATIBILITY:

linux-5.0 folder applies to kernel versions from 5.0 to 5.0.5

## BUILD INSTRUCTIONS:

- Download and install the kernel source
- Select the appropriate patch subfolder based on the selected kernel version
- Apply the relevant patches to your kernel sources
- Compile and install the kernel

## SETUP AND CONFIGURATION:

The patch enable the selection of the desired video mode during the boot process.
The parameters must be provided to your boot loader (grub, syslinux, ...) and appended to your kernel parameters

You can specify "640x480" or "800x600" resolution at boot by adding either "video=VGA-1:640x480ec" or "video=VGA-1:800x600ez" to the kernel line.

- "VGA-1" is the name of the video connector (see the kernel documentation or xrandr utility output for more info)
- 'e' letter is needed to switch on and enable the output connector
- 'c' letter or 'z' letter are used to select respectively 15KHz or 25KHz (default is 31kHz)

E.g. for syslinux.cfg:

```
append root=/dev/sda1 rw vga=785 <...other parameters...> video=VGA-1:640x480ec
```

## SOURCES:

1. MaKoTo, https://burogu.makotoworkshop.org/index.php?post/2012/12/26/borne-arcade-40
2. Ansa89, https://github.com/Ansa89/linux-15khz-patch
