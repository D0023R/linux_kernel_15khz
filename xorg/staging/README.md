# Experimental prerelease patches

These variants are separate from the [stable patch library](../README.md). They are source-checked experiments, with no build or hardware-test claim.

| Exact prerelease | Patch | Verification date |
| --- | --- | --- |
| Xorg Server 26.0.99.903, generic modesetting driver | [MIN192 patch](xserver-26.0.99.903/01_xorg_low_resolution_min_192x192.patch) | 2026-11-01 |

The exact upstream source still sets a `320x200` minimum. This patch lowers it to `192x192`, using a hunk starting at line 3938. It preserves the same code change as the stable `xserver-21.1` patch while naming the prerelease source and matching its line numbers.

Apply only this variant when evaluating the stated prerelease; it replaces the stable MIN192 variant for that source, rather than following it in a patch series. From the pristine `xorg-server-26.0.99.903` source directory:

```sh
patch --dry-run --fuzz=0 -p1 < /path/to/linux_kernel_15khz/xorg/staging/xserver-26.0.99.903/01_xorg_low_resolution_min_192x192.patch
patch --fuzz=0 -p1 < /path/to/linux_kernel_15khz/xorg/staging/xserver-26.0.99.903/01_xorg_low_resolution_min_192x192.patch
```

These usage examples require GNU patch. The recorded maintenance check used Git, verified exact context independently, and compared the complete resulting source file. See [validation details](../README.md) and the [prerelease source record](../docs/prerelease-source-verification.json) for archive and file hashes, commands, and results.

No applicability claim is made for 26.0.99.901, 26.0.99.902, future prereleases, or a future stable 26.x release. Each needs its own source review. Before proposing stable coverage, verify the actual stable archive and report any available build and runtime tests separately.
