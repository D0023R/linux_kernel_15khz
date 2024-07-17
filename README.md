# This repository contains the Linux Kernel 15kHz video patches

The provided kernel patches enable the 15kHz video output with additional features for Linux.

## PATCH LIBRARY CONTENT

**Current:** 

- *Mainline* release: **6.10**
- *Stable* release: **6.9.9**
- *Longterm* release: **6.6.40**
- *Longterm* release: **6.1.99**

**Untested experimental:** 

- *Next* release: **-**

| Filename                                           | Description                                                                             |
| -------------------------------------------------- | --------------------------------------------------------------------------------------- |
| 01_linux_15khz.patch                               | main patch for 15 kHz support                                                           |
| 02_linux_15khz_interlaced_mode_fix.patch           | necessary for radeon driver, fix the vertical blank interrupt                           |
| 03_linux_15khz_dcn1_dcn2_interlaced_mode_fix.patch | necessary for amdgpu driver, enable interlaced mode on standalone graphic cards and APU |
| 04_linux_15khz_dce_interlaced_mode_fix.patch       | necessary for amdgpu driver, enable interlaced mode on standalone graphic cards and APU |
| 05_linux_15khz_amdgpu_pll_fix.patch                | necessary for amdgpu driver, fix PLL calculation                                        |
| 06_linux_switchres_kms_drm_modesetting.patch       | KMS modesetting manipulation for X-less switchres KMS usage, groovyarcade kms enabler   |
| 07_linux_15khz_fix_ddc.patch                       | kernel 6.7+ only, fix kernel oops when probing DDC and no adapter is connected           |

### KERNEL COMPATIBILITY

For the latest stable release, please use the folder named `Linux-X.Y` from the root folder which is corresponding to your kernel version.
The repository will now be updated to the mainline stable releases only. Older kernel released will be available inside the `unmaintained` folder.

## BUILD INSTRUCTIONS

- Download and install the kernel source (https://kernel.org/)
- Select the appropriate patch sub-folder based on your kernel version
- Apply the relevant patches to the kernel source
- Compile and install the kernel

## SETUP AND CONFIGURATION

The patch enable the selection of the desired video mode during the boot process.
The parameters must be provided as kernel parameters via your boot loader (grub, syslinux, ...).

### Since kernel 5.6

You can specify interlace "640x480" or progressive "320x240" resolution at boot by adding either `video=VGA-1:640x480ieS` or `video=VGA-1:320x240eS` to the kernel line.

- "VGA-1" is the name of the video connector (see the kernel documentation or xrandr utility output for more info)
- parameter 'e' letter is needed to enable and activate the output connector
- parameter 'S' letter tells to use the low dotclock resolutions. As for now, resolutions are hard coded, here is a list of them. Use the exact resolution name ('i' letter means interlaced and is part of the name)

```
  - 15kHz modes:
    - "320x240"     /* 320x240 60.00 Hz */
    - "384x288"     /* 384x288 50.00 Hz */
    - "640x240"     /* 640x240 60.00 Hz */
    - "640x480i"    /* 640x480 60.00 Hz */
    - "648x480i"    /* 648x480 60.00 Hz */
    - "720x480i"    /* 720x480 59.95 Hz */
    - "768x576i"    /* 768x576 50.00 Hz */
    - "800x576i"    /* 800x576 50.00 Hz */
    - "1280x480i"   /* 1280x480 60.00 Hz */
  - 25kHz modes:
    - "512x384"     /* 512x384 58.59 Hz */
    - "800x600i"    /* 800x600 60.00 Hz */
    - "1024x768i"   /* 1024x768 50.00 Hz */
    - "1024x240"    /* 1024x240 60.00 Hz */
    - "1280x240"    /* 1280x240 60.00 Hz */
  - 31kHz modes:
    - "640x480"     /* 640x480 60.00 Hz */
```

E.g. for syslinux.cfg:

```
append root=/dev/sda1 rw vga=785 <...other parameters...> video=VGA-1:640x480ieS
```

### Before kernel 5.6

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
(Note: starting from kernel 4.15 the drm_kms_helper.edid_firmware parameter has been deprecated in favor of the drm.edid_firmware parameter. Backward compatibility for drm_kms_helper.edid_firmware is still present in kernel 5.3)

Put your edid custom file in the /lib/firmware/edid directory and add the following in your boot manager configuration on the kernel parameter line.

```
append root=/dev/sda1 <...other parameters...> video=VGA-1:e drm.edid_firmware=VGA-1:edid/<edid_filename>
```

The parameter video=VGA-1:e is needed to enable the connector. The drm.edid_firmware=VGA-1:edid/<edid_filename> parameter sets the custom EDID on the connector.  

NOTES:  
    On previous kernel use drm_kms_helper.edid_firmware instead of drm.edid_firmware.  
    If you are using early KMS with initramfs, the custom EDID file must be included or the kernel will not find it.  
    Using the custom EDID method, X server will start in the EDID defined configuration. No need to specify any modeline.  

### Optional for the time being (staging folder, on-hold)

- linux_15khz_scanoutpos.patch (enable scanline position reporting to user space, will be required to support future groovymame synchronisation library)

### Deprecated since kernel 5.6

- ati_9200_pllfix.patch (only required to support ATI 9200 card model)
- arcadevga_3000.patch (only required to support ARCADEVGA 3000 card model)

## CONTRIBUTORS:

- Substring, https://github.com/substring
- Calamity, https://github.com/antonioginer

## SOURCES/HISTORY:

1. MaKoTo, https://burogu.makotoworkshop.org/index.php?post/2012/12/26/borne-arcade-40
2. Ansa89, https://github.com/Ansa89/linux-15khz-patch
