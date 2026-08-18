# SM-F956B / F956BXXS4DZG3 Qualcomm ABL derivation

This note closes the physical-load-address gate for the Galaxy Z Fold6 `SM-F956B` build `F956BXXS4DZG3`. The inputs come from the matching DZG3 BL archive supplied for this port; no S928U load-address value is copied as evidence.

## Input provenance

| Object | SHA-256 |
| --- | --- |
| supplied BL `.tar.md5` | `b510e3690daec202b68913af1cbf3fcb7889419e20ee1176f5a3688492cfab58` |
| `abl.elf.lz4` | `e025f37c4f1866ab32737458e9305103fd7dae0c83554907c382f601b7167fa5` |
| decompressed `abl.elf` | `f3bf21658f39d60a661b34455549b6440dcded189679cd1c630b6649b5e74561` |
| decompressed ABL GUID-defined FV payload | `2b58dfa643caa5fb669cf649f9fa284e4a932f9c4facf6ad4744e4d6a5d2f65a` |
| extracted `LinuxLoader` PE32+ image | `167638c1c214814e27d3a865d6805c0d1fd58992ddf96e906fab28a8eb821c81` |
| decompressed `xbl_config.elf` | `88b61abc2f509c6b31e0d747c1301d4c5a0e42a10447ebe69ebc33cd6e2eaae2` |
| post-DDR XBL DTB | `afa980cacf2e2dc34e334ae3bdd7f9040d421e028fdd4eed50ea3f5576cd2fb9` |

The outer ABL is an ELF32 ARM container. Its firmware volume begins at file offset `0x1000`; the first GUID-defined section contains an LZMA-alone payload. Decompression exposes the UEFI firmware volume containing the FFS file named `LinuxLoader` with GUID `f536d559-459f-48fa-8bbc-43b554ecae8d`. The extracted image is an AArch64 PE32+ EFI application.

## LinuxLoader ARM64 load constants

The exact Fold6 `LinuxLoader` contains the four load constants at PE RVA/file offset `0x0d76e8`:

```text
0x0d76e8: 0x00080000
0x0d76ec: 0x05600000
0x0d76f0: 0x03c00000
0x0d76f4: 0x00008000
```

The selection code is at RVAs `0x17678` through `0x176c0`. For an ARM64 Image, the `ARMd` header test keeps the ARM64 path selected and loads the first pair:

```text
kernel offset = 0x00080000
kernel region = 0x05600000
```

The relevant sequence constructs the kernel load base from the lowest RAM base and the selected offset, then stores the load-base/region-end pair for the later boot path.

The target boot Image itself has:

```text
text_offset = 0x0
image_size  = 0x026f0000
magic       = ARMd
```

Therefore the final entry calculation does not add a nonzero Image `text_offset`.

## Lowest DDR base cross-check

The matching BL archive also contains `xbl_config.elf`. Its post-DDR DTB has:

```text
/soc/memorymap
  #address-cells = <2>
  #size-cells = <2>

/soc/memorymap/memory@80000000
  device_type = "memory"
  reg = <0x0 0x80000000 0x0 0x016e0000>
```

This independently fixes the lowest RAM base used by the Qualcomm LinuxLoader path at `0x80000000`.

## Final physical-load constants

For `SM-F956B / F956BXXS4DZG3`:

```c
#define P0_PHYS_OFFSET       0x80000000ULL
#define P0_KERNEL_PHYS_LOAD  0x80080000ULL
```

The second value is target-derived as:

```text
0x80000000 + 0x00080000 + text_offset(0)
= 0x80080000
```

The resulting constants are now present in `src/targets/q6q-F956BXXS4DZG3/target.h`.

## Status

The static target profile now has target-proven kernel symbols/layouts, P0 fingerprints, KASLR/slide constants, target-specific `nfnetlink_log` string offset, and Qualcomm physical load addresses. The remaining work is payload compilation, KernelSU exact-vermagic build/static import audit, and hardware validation.
