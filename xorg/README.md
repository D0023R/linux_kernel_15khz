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
| `xserver-1.20/` | Xorg Server 1.20.14 | `hw/xfree86/drivers/modesetting/drmmode_display.c` | Exact source context and patch application verified |
| `xserver-21.1/` | Xorg Server 21.1.24 | `hw/xfree86/drivers/modesetting/drmmode_display.c` | Exact source context and patch application verified |
| `xf86-video-amdgpu-19.1/` | xf86-video-amdgpu 19.1.0 | `src/drmmode_display.c` | Exact source context and patch application verified |
| `xf86-video-amdgpu-21.0/` | xf86-video-amdgpu 21.0.0 | `src/drmmode_display.c` | Exact source context and patch application verified |
| `xf86-video-amdgpu-22.0/` | xf86-video-amdgpu 22.0.0 | `src/drmmode_display.c` | Exact source context and patch application verified |
| `xf86-video-amdgpu-23.0/` | xf86-video-amdgpu 23.0.0 | `src/drmmode_display.c` | Exact source context and patch application verified |
| `xf86-video-amdgpu-25.0/` | xf86-video-amdgpu 25.0.0 | `src/drmmode_display.c` | Exact source context and patch application verified |

The version-family layout follows the same principle as the kernel patch library: select the directory matching the source revision you are building. The patch purpose and filename remain consistent while source-specific hunks can evolve with upstream changes.

A source-verified entry is not the same as a runtime-tested distribution build. Distribution backports or downstream modifications can change patch context, so always perform a dry run against the exact source package you intend to build.

On **2026-09-18**, all seven patches were checked against the complete target files from the official release archives listed in [docs/source-verification.json](docs/source-verification.json). Each hunk matches at its stated line, passes `git apply --check`, and applies successfully. Comparing the complete resulting file confirms that only `320, 200` changes to `192, 192` in `xf86CrtcSetSizeRange()`.

This refresh corrects missing blank-line context in five patches and updates hunk line numbers in all seven. It preserves the MIN192 code change. The verification record includes archive, source-file, and patch SHA-256 hashes. These are source/application checks; this refresh adds no new build or hardware-test claims.

## Applying a patch

Enter the extracted source tree for the component and run `patch -p1` with the matching patch file.

Example for Xorg Server 21.1:

```bash
cd xorg-server-21.1.24
patch --dry-run --fuzz=0 -p1 < /path/to/linux_kernel_15khz/xorg/xserver-21.1/01_xorg_low_resolution_min_192x192.patch
patch --fuzz=0 -p1 < /path/to/linux_kernel_15khz/xorg/xserver-21.1/01_xorg_low_resolution_min_192x192.patch
```

Example for xf86-video-amdgpu 25.0:

```bash
cd xf86-video-amdgpu-25.0.0
patch --dry-run --fuzz=0 -p1 < /path/to/linux_kernel_15khz/xorg/xf86-video-amdgpu-25.0/01_amdgpu_low_resolution_min_192x192.patch
patch --fuzz=0 -p1 < /path/to/linux_kernel_15khz/xorg/xf86-video-amdgpu-25.0/01_amdgpu_low_resolution_min_192x192.patch
```

These examples use GNU patch. Investigate any failed hunks or offsets against the exact source package. Compile and package the component using the normal procedure for your distribution after the patch applies cleanly.

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

Check both the overall `Screen 0` dimensions and the connector's active mode. For a single-output `256x224` test, the expected screen size is `current 256 x 224`. Lowering the minimum permits this framebuffer size; it does not force applications to request it or shrink a desktop that must accommodate other active outputs.

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

The current patches were refreshed against the exact official source baselines listed above. Older variants should be moved to `xorg/unmaintained/` when their maintenance coverage ends, with the last checked source baseline recorded.

## Community maintenance

**Maintainer:** Rion ([Redemp](https://github.com/Redemp)).

The maintained patch library is [xorg-15khz-crt-patches](https://github.com/Redemp/xorg-15khz-crt-patches). Rion checks upstream Xorg Server and xf86-video-amdgpu releases monthly and submits verified corrections or compatibility updates here through follow-up pull requests when needed. Maintenance of these Xorg patches is handled separately from D0023R's kernel patch updates.

This refresh synchronizes the seven patch files with [maintenance commit da9968846a35cea57504a2b9d2fbfa88bb183bfa](https://github.com/Redemp/xorg-15khz-crt-patches/tree/da9968846a35cea57504a2b9d2fbfa88bb183bfa).

Patch selection follows the Xorg component and source version, not the Linux kernel version. See the maintained library's [contribution guide](https://github.com/Redemp/xorg-15khz-crt-patches/blob/main/CONTRIBUTING.md) for source verification and test-report requirements.

## Xorg patch license and attribution

Rion's original Xorg patch contributions and documentation in this directory are available under the [MIT License](LICENSE). Upstream source context retains its original attribution and permission terms; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
