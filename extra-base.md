
# Kernel Configuration and Optimization References

Below is kernel options, that was enabled in kernels like CachyOS, Xanmod github kernels compared to NixOS linux kernel ( even xanmod variation ) by default.

## Useful Links
- [CachyOS Bore Config](https://github.com/CachyOS/linux-cachyos/blob/master/linux-cachyos-bore/config) - CachyOS config for reference.
- [CachyOS Hardened Config](https://github.com/CachyOS/linux-cachyos/blob/master/linux-cachyos-hardened/config) - CachyOS hardened config.
- [Xanmod Kernel Config](https://github.com/xanmod/linux/blob/6.9/CONFIGS/xanmod/gcc/config_x86-64-v3) - Xanmod git kernel config.

## NO_HZ: Reducing Scheduling-Clock Ticks

- [NO_HZ Documentation](https://www.kernel.org/doc/Documentation/timers/NO_HZ.txt)

```nix
NO_HZ_IDLE = yes; # Enabled in xanmod git kernel.
```

`NO_HZ_IDLE`: can reduce the number of scheduling-clock interrupts, thereby improving energy efficiency and reducing OS jitter. Reducing OS jitter is important for some types of computationally intensive high-performance computing (HPC) applications and for real-time applications.

## NO_HZ_FULL: Full dynticks system (Tickless)

- [NO_HZ_FULL](https://cateee.net/lkddb/web-lkddb/NO_HZ_FULL.html)
- [CPU Isolation Introduction](https://www.suse.com/c/cpu-isolation-introduction-part-1/)

```nix
NO_HZ_FULL = yes; # Used in CachyOS.
```
Adaptively try to shutdown the tick whenever possible, even when the CPU is running tasks. Typically this requires running a single task on the CPU. Chances for running tickless are maximized when the task mostly runs in userspace and has few kernel activity.

By default, without passing the `boot.kernelParams`, this behaves just like `NO_HZ_IDLE`, so we can safely use `NO_HZ_FULL` instead of `NO_HZ_IDLE`. 
I have are few ideas with this. More on my notes.

## Optimize kernel for x86_64-v* processors

```nix
GENERIC_CPU3 = yes; for x86_64-v3 processors. GENERIC_CPU4 for x86_64-v4 and so on.
```

## AMD cpu
- [Ryzen gentoo wiki](https://wiki.gentoo.org/wiki/Ryzen)
- [AMD microcode gentoo wiki](https://wiki.gentoo.org/wiki/AMD_microcode)

We can also enable compiler optimizations for AMD generation instead.

```nix
MZEN3 = yes; # for Zen3, for example. MZEN4 for zen4 and etc.
```

**CONFIG_FW_LOADER** = Firmware loading facility

> [!Warning]
> Building the firmware loading facility as a module, **{M} Firmware loading facility** i.e. `CONFIG_FW_LOADER=M` ( Like NixOS linux kernel do ), might prevent early microcode updating in most setups.
Source: gentoo wiki.

I personally didn't have any issues with this enabled as <M>.

```nix
FW_LOADER = yes;
```



## DMABUF

- [DMABUF Heaps](https://www.kernelconfig.io/config_dmabuf_heaps?arch=x86&kernelversion=6.9.7)
- [DMABUF Heaps CMA](https://www.kernelconfig.io/CONFIG_DMABUF_HEAPS_CMA?q=DMABUF_HEAPS_CMA&kernelversion=6.9.7&arch=x86)
- [DMABUF Move Notify](https://cateee.net/lkddb/web-lkddb/DMABUF_MOVE_NOTIFY.html)

 Who knows why "heaps" were disabled by default.

```nix
DMABUF_HEAPS = yes;
```

```nix
DMABUF_HEAPS_SYSTEM = yes; # Enabled in cachyos/xanmod git configs.
```  

```nix
DMABUF_HEAPS_CMA = yes; # Disabled in xanmod GitHub config. Enabled in CachyOS.
```  
```nix
DMABUF_MOVE_NOTIFY = yes; # Security. Disabled in CachyOS kernel, probably because of : "due to inconsistent execution context and memory management between drivers". Enabled in xanmod GitHub config.
```

## ZSWAP

- [Z3fold Documentation](https://www.kernel.org/doc/html/v5.0/vm/z3fold.html)
- [Zsmalloc Documentation](https://www.kernel.org/doc/html/v5.5/vm/zsmalloc.html)

```nix
ZSWAP_SHRINKER_DEFAULT_ON = yes; # https://cateee.net/lkddb/web-lkddb/ZSWAP_SHRINKER_DEFAULT_ON.html. Good thing overall, no? Enabled in cachyos/xanmod GitHub config.
ZSWAP_COMPRESSOR_DEFAULT_LZ4 = yes; # CachyOS and NixOS kernel use zstd instead. Xanmod GitHub use LZ4.
ZSWAP_COMPRESSOR_DEFAULT = "lz4"; # above.
ZSWAP_ZPOOL_DEFAULT_Z3FOLD = yes; # NixOS xanmod and CachyOS using zmalloc. Xanmod GitHub kernel using z3fold.
ZSWAP_ZPOOL_DEFAULT = "z3fold"; # above.
Z3FOLD = yes; # above.
ZSWAP_DEFAULT_ON = yes; # enabled in CachyOS.
ZSMALLOC_STAT = yes; # enabled in CachyOS. Monitoring and debugging stuff.
```

## ZRAM

> [!NOTE]
> ZRAM by default is enabled as a module which is correct because it allows changing the number of zram devices without rebooting, by deactivating zram devices and re-loading the module with new parameters. 

> Options below were disabled in NixOS xanmod, but xanmod GitHub and CachyOS Bore + Hardened enable them.

```nix
ZRAM_TRACK_ENTRY_ACTIME = yes;
ZRAM_MEMORY_TRACKING = yes; # Track ZRAM block status.
```

> Enabled in xanmod GitHub. LZO instead of "zstd"

```nix
ZRAM_DEF_COMP_LZORLE = yes; #
ZRAM_DEF_COMP = "lzo-rle";
```

## TMPFS
- [TMPFS_INODE64](https://www.kernel.org/doc/html/v5.0/vm/z3fold.html)

```nix
TMPFS_INODE64 = yes; 
TMPFS_QUOTA = yes; # QUOTA support.
```

