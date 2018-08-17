# Buildroot - Making Embedded Linux Easy

Buildroot is a set of Makefiles and patches that simplifies and automates the process of building a complete and bootable Linux environment for an embedded system, while using cross-compilation to allow building for multiple target platforms on a single Linux-based development system. Buildroot can automatically build the required cross-compilation toolchain, create a root file system, compile a Linux kernel image, and generate a boot loader for the targeted embedded system, or it can perform any independent combination of these steps. For example, an already installed cross-compilation toolchain can be used independently, while Buildroot only creates the root file system.

See [buildroot@github.com: pine64](https://github.com/buildroot/buildroot/tree/master/board/pine64/pine64)
for instructions how to build a pine64 OS image.

## Pine64 Configuration File

**dot.config.pine64** provides a configuration derived from **pine64_defconfig** which comes with buildroot.

You may replace the configuration file **.config** in buildroot's base directory with this file and run the build.


## Pine64 Plus Board Configuration

The patch files **0001-default-configuration-pine64_plus-board.patch** and **0002-bump-kernel-version-to-4.17.15.patch**
were created on top of commit:


```
commit 65c47ce0a8d646211a5222035a821056896a5886
Author: Bernd Kuhls <bernd.kuhls@t-online.de>
Date:   Wed Aug 15 22:17:21 2018 +0200

    linux: bump default to version 4.17.15

    Signed-off-by: Bernd Kuhls <bernd.kuhls@t-online.de>
    Signed-off-by: Peter Korsgaard <peter@korsgaard.com>
```

They may as well apply on other versions.

The patch introduces a new board type, the **Pine64 Plus**.

Run
```
$ make list-defconfigs | grep pine
```

to show all configurations for a **Pine64** based board:

```
  pine64_defconfig                    - Build for pine64
  pine64_plus_defconfig               - Build for pine64_plus
  pine64_sopine_defconfig             - Build for pine64_sopine

```

Select the configuration for **Pine64 Plus** and call make to run the build:

```
$ make pine64_plus_defconfig
$ make

```
