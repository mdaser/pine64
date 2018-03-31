# Build Mainline Kernel for Pine64

The full support for the Pine64 board was finally added to kernel version 4.15; see **Sunxi Pine64 Wiki - Linux Kernel**.

In order to build a kernel of version 4.15 (or newer), use a configuration file extracted from a working image (e.g. the Ubuntu image with the 3.6.64 kernel by longsleep).

## Clone the kernel sources:
```
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
$ cd linux
$ git checkout ...
```

## Install a Cross Toolchain for arm64/aarch64

Download a cross toolchanin (e.g. by Linaro, see **[LINA1]**), unpack it, and set up the environment:
```
$ wget ...
$ export ARCH="arm64"
$ export CROSS_COMPILE="aarch64-linux-gnu-"
$ export SYSROOT=".../gcc-linaro-5.5.0-2017.10-x86_64_aarch64-linux-gnu/aarch64-linux-gnu"
```

Add `.../gcc-linaro-5.5.0-2017.10-x86_64_aarch64-linux-gnu/bin` to your **PATH** variable; preferably at the beginning.

## Configurations

* config-3.10.65-longsleep: Kernel configuration from longsleep Ubuntu image
* config-4.14.26: from ??
* config-4.15.xx: configuration after **make oldconfig**

## Make oldconfig

Copy one of the above configuration files to the kernel source tree and build the **oldconfig** target. This step will take all the settings of the initial confuguraton and add them to the curretn kernel version. You may have to acknowledge a few settings which were not present in the bas kernel version.

```
$ cp .../config-4.14.26 .
$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- oldconfig
```

## Build the Kernel

Build the kernel:
```
$ time make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image
```

You will find the kernel Image in: `arch/arm64/boot/Image`

Build and install the kernel modules:
```
$ time make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- modules
$ mkdir __pine64
$ time make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=./__pine64 modules_install
```

You will find the kernel modules in `__pine64/lib/modules/<kernel version>`

Build and install the device trees:
```
$ time make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- dtbs
```

You will find the device trees in:
* `./arch/arm64/boot/dts/allwinner/sun50i-a64-pine64-plus.dtb`
* `./arch/arm64/boot/dts/allwinner/sun50i-a64-pine64.dtb`
