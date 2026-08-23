# Xorg low-resolution patches

This directory contains X11/Xorg userspace patches for native low-resolution CRT modes.

These patches are separate from the Linux kernel 15 kHz patches in the repository. They do not change DRM/KMS mode validation, KMSRAW operation, pixel-clock limits, interlace support, or AMDGPU kernel behavior. They only lower the minimum Xorg screen-size range advertised by the active Xorg display driver.

## What the patch changes

Both Xorg's generic `modesetting` DDX and the dedicated `xf86-video-amdgpu` DDX contain a call equivalent to:

```c
xf86CrtcSetSizeRange(pScrn, 320, 200,
                     mode_res->max_width,
                     mode_res->max_height);
```

The patches in this directory change the minimum from `320 x 200` to `192 x 192`:

```c
xf86CrtcSetSizeRange(pScrn, 192, 192,
                     mode_res->max_width,
                     mode_res->max_height);
```

`192 x 192` is a deliberately conservative tested userspace floor. It allows common native CRT widths such as `256x224` while avoiding an unnecessarily unrestricted minimum. Lowering this range does not force a normal monitor to use a low-resolution mode; it only allows Xorg to accept a smaller root screen when such a mode is deliberately selected.

## Why there are two patch families

The correct patch depends on which Xorg display driver your system actually loads.

- **Generic Xorg modesetting DDX**: the code is part of `xorg-server`; use an `xserver-*` patch.
- **Dedicated AMDGPU Xorg DDX**: the code is part of `xf86-video-amdgpu`; use an `xf86-video-amdgpu-*` patch.

In practice, select the patch family from the driver Xorg actually loads:

- `modesetting_drv.so` -> patch the Xorg Server `modesetting` DDX using the matching `xserver-*` directory.
- `amdgpu_drv.so` -> patch the dedicated AMDGPU Xorg DDX using the matching `xf86-video-amdgpu-*` directory.

This distinction is important: patching `xorg-server` does not alter the same minimum when Xorg is using `amdgpu_drv.so`, and patching `xf86-video-amdgpu` does not alter the generic `modesetting_drv.so` path.

## Identify the active Xorg driver

Check the Xorg log before selecting a patch:

```bash
grep -E 'modesetting_drv.so|amdgpu_drv.so' /var/log/Xorg.0.log
```

Typical results:

```text
modesetting_drv.so  -> use xserver-X.Y/
amdgpu_drv.so       -> use xf86-video-amdgpu-X.Y/
```

If your distribution stores the Xorg log elsewhere, query that log instead. Also verify the installed package version before applying a patch.

## Patch library content

| Directory | Component / source baseline | Patched source file | Validation status |
| --- | --- | --- | --- |
| `xserver-1.20/` | Xorg Server 1.20 family; authored against 1.20.14 | `hw/xfree86/drivers/modesetting/drmmode_display.c` | Source location verified; same call also checked in 1.20.11 |
| `xserver-21.1/` | Xorg Server 21.1 family; authored against 21.1.24 | `hw/xfree86/drivers/modesetting/drmmode_display.c` | Source verified against 21.1.24; MIN192 behavior runtime-tested through `modesetting_drv.so` |
| `xf86-video-amdgpu-19.1/` | xf86-video-amdgpu 19.1.0 | `src/drmmode_display.c` | Source verified |
| `xf86-video-amdgpu-21.0/` | xf86-video-amdgpu 21.0.0 | `src/drmmode_display.c` | Source verified |
| `xf86-video-amdgpu-22.0/` | xf86-video-amdgpu 22.0.0 | `src/drmmode_display.c` | Source verified |
| `xf86-video-amdgpu-23.0/` | xf86-video-amdgpu 23.0.0 | `src/drmmode_display.c` | Source verified |
| `xf86-video-amdgpu-25.0/` | xf86-video-amdgpu 25.0.0 | `src/drmmode_display.c` | Source verified and runtime-tested through `amdgpu_drv.so` |

The version-family layout follows the same principle as the kernel patch library: select the directory matching the source revision you are building. The patch purpose and filename remain consistent while source-specific hunks can evolve with upstream changes.

A source-verified entry is not the same as a runtime-tested distribution build. Distribution backports or downstream modifications can change patch context, so always perform a dry run against the exact source package you intend to build.

## Applying a patch

Enter the extracted source tree for the component and run `patch -p1` with the matching patch file.

Example for Xorg Server 21.1:

```bash
cd xorg-server-21.1.24
patch --dry-run -p1 < /path/to/linux_kernel_15khz/xorg/xserver-21.1/01_xorg_low_resolution_min_192x192.patch
patch -p1 < /path/to/linux_kernel_15khz/xorg/xserver-21.1/01_xorg_low_resolution_min_192x192.patch
```

Example for xf86-video-amdgpu 25.0:

```bash
cd xf86-video-amdgpu-25.0.0
patch --dry-run -p1 < /path/to/linux_kernel_15khz/xorg/xf86-video-amdgpu-25.0/01_amdgpu_low_resolution_min_192x192.patch
patch -p1 < /path/to/linux_kernel_15khz/xorg/xf86-video-amdgpu-25.0/01_amdgpu_low_resolution_min_192x192.patch
```

Compile and package the component using the normal procedure for your distribution after the patch applies cleanly.

## Runtime verification

After installing the rebuilt component and restarting Xorg, check the minimum screen size:

```bash
xrandr | head -n 1
```

The expected result contains:

```text
Screen 0: minimum 192 x 192, ...
```

Then verify a native low-resolution mode such as `256x224` using the same CRT/Switchres configuration used for the unpatched comparison.

## Initial test motivation and results

The MIN192 change came from investigating native-width CRT presentation and tearing/black-screen behavior rather than from a theoretical minimum-size cleanup.

The initial A/B testing established that Xorg's stock `320x200` floor can prevent the X root framebuffer from following native CRT modes below 320 pixels wide. With the minimum lowered to `192x192`, a native `256x224` Xorg screen becomes possible instead of retaining a wider minimum framebuffer.

The change was independently exercised through both relevant Xorg paths:

- generic `modesetting_drv.so`;
- dedicated `amdgpu_drv.so` from `xf86-video-amdgpu`.

During the wider CRT investigation, native `256x224` RetroArch presentation became smooth when Xorg could actually use the native screen width. A long-standing black-screen transition seen in *Castlevania: Symphony of the Night* with `dotclock_min=0` also disappeared with the MIN192 Xorg change; horizontally multiplying the mode through a higher `dotclock_min` had previously masked the problem.

These observations are why this patch should be treated as a general Xorg native-low-resolution fix/workaround rather than as a Navi10- or RDNA1-specific patch.

## Scope and limitations

This patch does **not**:

- add 15 kHz modes to the kernel;
- bypass AMDGPU minimum pixel-clock restrictions;
- enable interlaced modes in the kernel;
- change Switchres KMS/KMSRAW behavior;
- force every application to render at the X root size;
- guarantee that every historical point release in a version family has identical downstream source context.

It only changes the Xorg userspace minimum screen-size range from `320x200` to `192x192` in the selected DDX.

## Upstream source references

Release archives:

- Xorg Server: https://www.x.org/releases/individual/xserver/
- Xorg video drivers: https://www.x.org/releases/individual/driver/

The initial patch set was checked against the corresponding tagged or distribution-hosted source baselines before being added here. Older variants should be moved to `xorg/unmaintained/` when they are no longer maintained, following the repository's kernel-version policy.
