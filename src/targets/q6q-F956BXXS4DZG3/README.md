# q6q / F956BXXS4DZG3

Exact-firmware target for Samsung Galaxy Z Fold6 `SM-F956B`, build `F956BXXS4DZG3`.

Static derivation status: **complete** for the supplied DZG3 AP boot image and matching BL archive. The target header is based on target-derived evidence rather than sibling-device assumptions.

Verified target facts:

- kernel release: `6.1.145-android14-11-33418572-abF956BXXS4DZG3`
- raw kernel SHA-256: `63bc02e54747a0d85bba41effe619fa25ca763dc74af1fd09678fb6795983970`
- BTF SHA-256: `8415104c012e18942b18bcb52f401075cb6b92df837b9552a8c11070d65efe56`
- compact Android 6.1 `rt_mutex_waiter` ABI
- trace event ID: `106`
- worker caller offset: `0x000db1a0`
- pselect word shift: `3`
- target-generated P0 fingerprint table
- F956B-specific `nfnetlink_log` Image offset: `0x016a61e6`
- `P0_PHYS_OFFSET = 0x80000000`
- `P0_KERNEL_PHYS_LOAD = 0x80080000`

The physical-load constants are independently derived from the matching DZG3 Qualcomm `LinuxLoader` and XBL post-DDR device tree. They are not copied from E3Q.

See `docs/SM-F956B-F956BXXS4DZG3.md` for the kernel/profile derivation and `docs/SM-F956B-F956BXXS4DZG3-ABL.md` for the Qualcomm ABL/XBL derivation.

The draft PR is validated by `.github/workflows/validate-f956b-dzg3-port.yml` from the default branch. It compiles the exact target payload and exact-vermagic Samsung KernelSU module before any hardware test.

Hardware exploit execution and KernelSU late-load remain separate validation steps; this checked-in profile is not yet a claim of device-tested root.
