# q6q / F956BXXS4DZG3

Static AP/kernel analysis is complete for the supplied exact DZG3 boot image.

Verified target facts:

- kernel release: `6.1.145-android14-11-33418572-abF956BXXS4DZG3`
- raw kernel SHA-256: `63bc02e54747a0d85bba41effe619fa25ca763dc74af1fd09678fb6795983970`
- BTF SHA-256: `8415104c012e18942b18bcb52f401075cb6b92df837b9552a8c11070d65efe56`
- compact Android 6.1 `rt_mutex_waiter` ABI
- trace event ID: `106`
- worker caller offset: `0x000db1a0`
- pselect word shift: `3`
- P0 fingerprint generated from DZG3 and byte-identical to the checked-in E3Q DZF2 table
- F956B-specific `nfnetlink_log` Image offset: `0x016a61e6`

`target.h` is intentionally not present yet. The matching `BL_F956BXXS4DZG3...` Qualcomm `abl.elf` is required to prove `P0_PHYS_OFFSET` and `P0_KERNEL_PHYS_LOAD`. Do not substitute E3Q or another Snapdragon target's physical-load constants.

See `docs/SM-F956B-F956BXXS4DZG3.md` for the full provenance and recovered offsets.