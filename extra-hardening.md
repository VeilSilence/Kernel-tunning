# Kernel Configuration for hardening.

Below is kernel options, that was enabled in kernels like CachyOS, Xanmod github kernels compared to NixOS linux kernel ( even xanmod variation ) by default.

I personally didn't test everything below, but I'm planning to do so.

## Useful Links
- [CachyOS Bore Config](https://github.com/CachyOS/linux-cachyos/blob/master/linux-cachyos-bore/config) - CachyOS config for reference.
- [CachyOS Hardened Config](https://github.com/CachyOS/linux-cachyos/blob/master/linux-cachyos-hardened/config) - CachyOS hardened config.
- [Xanmod Kernel Config](https://github.com/xanmod/linux/blob/6.9/CONFIGS/xanmod/gcc/config_x86-64-v3) - Xanmod git kernel config.
- [NixOS Kernel Config](https://github.com/NixOS/nixpkgs/blob/nixos-24.05/pkgs/os-specific/linux/kernel/common-config.nix) - NixOS kernel config. NixOS xanmod share this config with few changes.

### Extra Hardening.

> [!WARNING]
> Do not enable/set kernel/boot options below blindly.

- [Integrity Measurement Architecture (IMA)](https://wiki.gentoo.org/wiki/Integrity_Measurement_Architecture)
- [Extended Verification Module (EVM)](https://wiki.gentoo.org/wiki/Extended_Verification_Module)
- [Smack (Simplified Mandatory Access Control Kernel)](https://www.kernel.org/doc/html/v4.14/admin-guide/LSM/Smack.html)

## IMA - Integrity Measurement Architecture

> [!IMPORTANT]
> To enable IMA, the kernel options below need to be enabled and set the kernel boot option `lsm=integrity`. 
> This needs to be implemented very carefully.

```nix

INTEGRITY = yes;
IMA = yes;
IMA_MEASURE_PCR_IDX = freeform "10";
IMA_LSM_RULES = yes;
INTEGRITY_SIGNATURE = yes;
IMA_APPRAISE = yes;

```


> [!NOTE]
> For the reference, only the following kernel options are enabled in Cachyos-hardened:

```nix

- INTEGRITY_SIGNATURE = yes;
- INTEGRITY_ASYMMETRIC_KEYS = yes;
- INTEGRITY_TRUSTED_KEYRING = yes;
- INTEGRITY_PLATFORM_KEYRING = yes;
- INTEGRITY_MACHINE_KEYRING = yes;

```

> [!NOTE]
> All options, related to IMA that enabled in Xanmod Github config:

```nix

- IMA_KEXEC = yes;
- IMA_NG_TEMPLATE = yes;
- IMA_DEFAULT_TEMPLATE = "ima-ng";
- IMA_DEFAULT_HASH_SHA256 = yes;
- IMA_DEFAULT_HASH** = "sha256";  # or sha512
  - `IMA_DEFAULT_HASH_SHA512` is also an option.
- IMA_ARCH_POLICY = yes;
- IMA_APPRAISE_BOOTPARAM = yes;
- IMA_APPRAISE_MODSIG = yes;
- IMA_MEASURE_ASYMMETRIC_KEYS = yes;
- IMA_QUEUE_EARLY_BOOT_KEYS = yes;
- IMA_SECURE_AND_OR_TRUSTED_BOOT = yes;

```

## EVM - Extended Verification Module

With EVM, the Linux kernel can be used to validate security-sensitive extended attributes before allowing operations on the files. 

> [!Caution]
> Disabled in Cachyos-hardened. Gentoo also warns about enabling this.

 
In order to enable EVM:

```nix

EVM = yes;
TCG_TPM #enabled as a module already in nixos.

```
Other EVM related flags:
```nix

EVM_ATTR_FSUUID = yes;
EVM_EXTRA_SMACK_XATTRS = yes;  # if SMACK enabled. SMACK is disabled by default.
EVM_ADD_XATTRS = yes;

```

  - INTEGRITY **required** and `boot.kernelParams "evm=fix"` on the first boot.
  - Then, you need to set up keys in TPM. 
  - More details in the [Gentoo wiki](https://wiki.gentoo.org/wiki/Extended_Verification_Module).


## SMACK - Simplified Mandatory Access Control Kernel

Smack is the Simplified Mandatory Access Control Kernel. Smack is a kernel based implementation of mandatory access control that includes simplicity in its primary design goals.

Smack is not the only Mandatory Access Control scheme available for Linux. Those new to Mandatory Access Control are encouraged to compare Smack with the other mechanisms available to determine which is best suited to the problem at hand.

Smack consists of three major components:

  - The kernel
  - Basic utilities, which are helpful but not required
  - Configuration data

Smack kernels use the CIPSO IP option. Some network configurations are intolerant of IP options and can impede access to systems that use them as Smack does.

Smack is used in the Tizen operating system. Please go to http://wiki.tizen.org for information about how Smack is used in Tizen.

Below is copied from Cachyos-hardened:

```nix
SECURITY_SMACK = yes;
SECURITY_SMACK_BRINGUP = yes;
SECURITY_SMACK_NETFILTER = yes;
SECURITY_SMACK_APPEND_SIGNALS = yes;
 DEFAULT_SECURITY_SMACK is not set. # Cachyos likely leaves this up to the user.
```

## `CONFIG_DEFAULT_SECURITY` comparison between different kernel configs

- [SELinux Introduction](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/security-enhanced_linux/chap-security-enhanced_linux-introduction#chap-Security-Enhanced_Linux-Introduction)
- [SECURITY_LOADPIN](https://cateee.net/lkddb/web-lkddb/SECURITY_LOADPIN.html)
- [SECURITY_SAFESETID](https://cateee.net/lkddb/web-lkddb/SECURITY_SAFESETID.html)

### Cachyos (hardened) Settings

```nix
 `CONFIG_DEFAULT_SECURITY_SELINUX` # is not set ( kernel related options are enabled, just not as "DEFAULT" )
 `CONFIG_DEFAULT_SECURITY_SMACK` # is not set ( above )
 `CONFIG_DEFAULT_SECURITY_TOMOYO` # is not set (  above )
 `CONFIG_DEFAULT_SECURITY_APPARMOR` # is not set (above )
```
  - **LOADPIN** - any files read through the kernel file reading interface (kernel modules, firmware, kexec images, security policy) can be pinned to the first filesystem used for loading. When enabled, any files that come from other filesystems will be rejected.
> Probably require more configuration than just "enabling" this option in kernel.

 ```nix
CONFIG_SECURITY_LOADPIN = yes;
CONFIG_SECURITY_LOADPIN_ENFORCE = yes;
```

  -  **SafeSetID** - is an LSM module that gates the setid family of syscalls to restrict UID/GID transitions from a given UID/GID to only those approved by a system-wide whitelist.

```nix
 CONFIG_SECURITY_SAFESETID = yes;
```

  - **DAC** - Unix Discretionary Access Controls. Not much info outside RedHat "SELinux introduction". Most distros use AppArmor as "DEFAULT" instead.
```nix
CONFIG_DEFAULT_SECURITY_DAC = yes;
```

```nix

CONFIG_SECURITY_LOCKDOWN_LSM = yes;
CONFIG_LSM = "landlock,lockdown,yama,integrity,bpf";
```

### Xanmod (GitHub config) Settings

```nix

 CONFIG_DEFAULT_SECURITY_SELINUX is not set # kernel related options are enabled, just not as "DEFAULT" 
 CONFIG_DEFAULT_SECURITY_SMACK is not set # same as above
 CONFIG_DEFAULT_SECURITY_TOMOYO is not set # same as above
 CONFIG_SECURITY_LOADPIN` is not set # disabled in kernel
 CONFIG_SECURITY_LOADPIN_ENFORCE is not set # above
 CONFIG_SECURITY_SAFESETID = yes;
 CONFIG_SECURITY_LOCKDOWN_LSM = yes;
 CONFIG_DEFAULT_SECURITY_APPARMOR = yes;
  CONFIG_DEFAULT_SECURITY_DAC is not set.
 CONFIG_LSM = "landlock,lockdown,yama,integrity,apparmor";
```

### Xanmod NixOS Settings

```nix

CONFIG_DEFAULT_SECURITY_TOMOYO is not set # disabled in the kernel also
CONFIG_DEFAULT_SECURITY_SELINUX is not set # disabled as "DEFAULT", but enabled in the kernel
CONFIG_DEFAULT_SECURITY_DAC is not set # disabled
CONFIG_SECURITY_SMACK is not set # disabled in the kernel also
CONFIG_SECURITY_LOADPIN is not set # disabled
CONFIG_SECURITY_LOADPIN_ENFORCE is not set # disabled
CONFIG_SECURITY_SAFESETID is not set # disabled
CONFIG_SECURITY_LOCKDOWN_LSM = yes;  # disabled
CONFIG_DEFAULT_SECURITY_APPARMOR = yes;
CONFIG_LSM = "landlock,lockdown,yama,loadpin,safesetid,selinux,smack,tomoyo,apparmor,bpf"; 
  
```

