# SM-F956B / F956BXXS4DZG3 porting record

This record tracks the exact-firmware port for the Galaxy Z Fold6 `SM-F956B` build `F956BXXS4DZG3`. Values from sibling Snapdragon devices or other Fold6 builds must not be reused unless independently verified against this exact kernel and bootloader.

## Stage 1: target identity

Status: **PARTIAL — runtime identity verified; firmware images still required**

| Field | Verified value |
| --- | --- |
| Model | `SM-F956B` |
| Device codename | `q6q` |
| Android | `16` |
| Android build ID | `BP4A.251205.006` |
| AP/build | `F956BXXS4DZG3` |
| Build fingerprint | `samsung/q6qxeea/q6q:16/BP4A.251205.006/F956BXXS4DZG3:user/release-keys` |
| Security patch | `2026-07-05` |
| SoC | `SM8650` |
| Kernel release | `6.1.145-android14-11-33418572-abF956BXXS4DZG3` |
| Verified boot state | `green` |
| vbmeta device state | `locked` |
| flash locked | `1` |

The 6.1.145 release makes the target worth checking for the CVE-2026-43499 rtmutex path, but kernel version alone is not evidence that Samsung did not backport the fix. Exploitability remains **unverified** until the exact raw kernel is inspected and/or the target profile passes hardware validation.

## Required firmware evidence

Before creating `src/targets/q6q-F956BXXS4DZG3/target.h`, freeze and hash the exact target inputs:

- AP: `boot.img.lz4`
- AP: `vendor_boot.img.lz4` (recommended for DTB/platform cross-checks)
- BL: `abl.elf.lz4`

Do not publish an exploit payload built by copying offsets from E3Q/S928U, Q7Q/F966, or another F956B build.

## Planned static derivation

1. Decompress `boot.img.lz4` and extract the raw ARM64 Image from the Android boot image.
2. Record SHA-256 for compressed boot, decompressed boot and raw kernel.
3. Recover `vmlinux.elf` with `vmlinux-to-elf` and validate the recovered symbol table.
4. Locate and validate the embedded BTF blob; derive all required structure layouts from target BTF raw output.
5. Derive the exploit symbol offsets from the target ELF, including ashmem/configfs/pipe/workqueue/root/SELinux/slide objects.
6. Derive `SLIDE_TRACEFS_EVENT_ID`, `SLIDE_TRACEFS_WORKER_CALLER_OFF`, `SLIDE_PSELECT_WORD_SHIFT`, and `SKB_DATA_DELTA` from this exact kernel.
7. Generate the 32-row P0 fingerprint table from the raw target Image and verify source readback.
8. Analyze the matching Qualcomm `abl.elf` to derive `P0_PHYS_OFFSET` and `P0_KERNEL_PHYS_LOAD`; do not inherit S928U values without proof.
9. Create `src/targets/q6q-F956BXXS4DZG3/{target.h,p0_fingerprint.h}` only after the above values are frozen.
10. Build a Samsung KDP/RKP/DEFEX KernelSU v3.2.5 module with exact vermagic `6.1.145-android14-11-33418572-abF956BXXS4DZG3`, then audit every undefined import against the recovered target ELF before late-load testing.

## Closest repository references

- `e3q-S928USQS6DZF2`: Snapdragon 8 Gen 3 / Android14-6.1 reference only; exact offsets and physical-load values are not portable.
- `q7q-F966USQU9BZDN`: foldable-family reference only; it is a different SoC generation/build and its values are not portable.

## Safety gate

No `target.h`, support-feed entry, release payload or KernelSU binary should be published for this target until the exact DZG3 images are hash-frozen and the static profile passes consistency checks. Hardware execution is a separate final validation step.
