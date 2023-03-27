# Buildroot - Making Embedded Linux Easy

Buildroot is a set of Makefiles and patches that simplifies and automates the process of building a complete and bootable Linux environment for an embedded system, while using cross-compilation to allow building for multiple target platforms on a single Linux-based development system.

Buildroot can automatically build the required cross-compilation toolchain, create a root file system, compile a Linux kernel image, and generate a boot loader for the targeted embedded system, or it can perform any independent combination of these steps.

For example, an already installed cross-compilation toolchain can be used independently, while Buildroot only creates the root file system.

See [buildroot@github.com: pine64](https://github.com/buildroot/buildroot/tree/master/board/pine64/pine64)
for instructions how to build a pine64 OS image.


At the time of this writing, buildroot supports the Pine64 board without extra modifications.

Run
```
$ make list-defconfigs | grep pine
```

to show all configurations for a **Pine64** based board:

```
  pine64_defconfig                    - Build for pine64
  pine64_sopine_defconfig             - Build for pine64_sopine

```

Select the configuration for **Pine64** and call make to run the build:

```
$ make pine64_plus_defconfig
$ make

```


You may tweak the settings for buildroot, the kernel, or uboot:
```
$ make menuconfig
$ make linux-menuconfig
$ make ubioot-menuconfig

```
