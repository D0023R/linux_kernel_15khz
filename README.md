# This repository contains the Linux kernel 15kHz video patches

The provided kernel patches enable the 15kHz video support with additional features for Linux.

## PATCH LIBRARY CONTENT:

- ati_9200_pllfix.diff (only required to support ATI 9200 card model)
- arcadevga_3000.diff (only required to support ARCADEVGA 3000 card model)
- linux_15khz.diff (main patch for 15 kHz support)
- linux_15khz_interlaced_mode_fix.diff (fix the vertical blank interrupt reporting to make interlaced resolution workings with groovymame)
- linux_15khz_scanoutpos.diff (optional, enable scanline position reporting to user space, will be required to support future groovymame synchronisation library)

## KERNEL COMPATIBILITY:

- linux-5.0 folder applies to 5.0 kernel versions.
- linux-5.1 folder applies to 5.1 kernel versions.
- linux-5.2 folder applies to 5.2 kernel versions.
- linux-5.3 folder applies to 5.3 kernel versions.

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

## Custom EDID method (no kernel patch required)

It is possible to set a custom EDID at boot via the drm module.  
(Note: in recent kernel the drm_kms_helper.edid_firmware parameter has been moved to the drm module. Backward compatibility for drm_kms_helper.edid_firmware is still present in kernel 5.3)

Put your edid custom file in the /lib/firmware/edid directory and add the following in your boot manager configuration on the kernel parameter line.
```
append root=/dev/sda1 <...other parameters...> video=VGA-1:e drm.edid_firmware=VGA-1:edid/<edid_filename>
```
The parameter video=VGA-1:e is needed to enable the connector. The drm.edid_firmware=VGA-1:edid/<edid_filename> parameter sets the custom EDID on the connector.  
    

NOTES:  
    On previous kernel use drm_kms_helper.edid_firmware instead of drm.edid_firmware.  
    If you are using an initramfs to initialize the kernel, the custom EDID file must be included or the kernel will not find it.  
    Using the custom EDID method, X server will start in the EDID defined configuration. No need to specify any modeline.  

## SOURCES:

1. MaKoTo, https://burogu.makotoworkshop.org/index.php?post/2012/12/26/borne-arcade-40
2. Ansa89, https://github.com/Ansa89/linux-15khz-patch
