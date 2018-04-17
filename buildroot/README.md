# Buildroot - Making Embedded Linux Easy

Buildroot is a set of Makefiles and patches that simplifies and automates the process of building a complete and bootable Linux environment for an embedded system, while using cross-compilation to allow building for multiple target platforms on a single Linux-based development system. Buildroot can automatically build the required cross-compilation toolchain, create a root file system, compile a Linux kernel image, and generate a boot loader for the targeted embedded system, or it can perform any independent combination of these steps. For example, an already installed cross-compilation toolchain can be used independently, while Buildroot only creates the root file system.

See [buildroot@github.com: pine64](https://github.com/buildroot/buildroot/tree/master/board/pine64/pine64)
for instructions how to build a pine64 OS image.

**dot.config.pine64** provides a configuration derived from **pine64_defconfig** which comes with buildroot.
