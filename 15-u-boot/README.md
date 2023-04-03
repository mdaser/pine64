# Das U-Boot

**Das U-Boot** (subtitled "the Universal Boot Loader" and often shortened to U-Boot) is an open source, primary boot loader used in embedded devices to package the instructions to boot the device's operating system kernel. It is available for a number of computer architectures, including 68k, ARM, Blackfin, MicroBlaze, MIPS, Nios, SuperH, PPC, RISC-V and x86.

The boot process consists of two steps initiated by the ROM monitor.
They are called **fsbl** and **ssbl**.

  * **fsbl** is provided by the **ARM Trusted Firmware**
  * **ssbl** is provided by **U-Boot**

The build process generates a single binary which will be flashed to an SD card.

## Build U-Boot

Select a toolchain which matches the Pine 64 board / A64 SoC, e.g. **aarch64-none-linux-gnu**.

### Build ARM Trusted Firmware

Clone the sources for  ARM Trusted firmware:

``` bash
$ git clone https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git
$ cd trusted-firmware-a/
$ git checkout master
```

Depending on the state of **master** you may have to check out a different version.

There are build instructions in *.../docs/plat/allwinner.rst*.

Select the default configuration for the board.

``` bash
$ make pine64_plus_defconfig
```

**TODO** change extra settings ... if required.

Finally, run the build.

``` bash
$ make CROSS_COMPILE=aarch64-none-linux-gnu- DEBUG=1 PLAT=sun50i_a64
```

You should find **bl31.bin** in *.../build/sun50i_a64/debug* or *.../build/sun50i_a64/release*.

### Build U-Boot

Clone the U-Boot sources:

``` bash
$ git clone https://gitlab.denx.de/u-boot/u-boot
$ cd u-boot
$ git checkout v2022.07
```

You may have to check out a different stable version.

There are build instructions in *.../doc/board/allwinner/sunxi.rst*.

Specify the paths to the **bl31.bin** and **SCP** files; built above.

``` bash
$ export BL31=../rusted-firmware-a/build/sun50i_a64/release/bl31.bin
$ export SCP=/dev/null
```

As we don't use SCP, set it to */dev/null*.

Select the default configuration for the board.

``` bash
$ make CROSS_COMPILE=aarch64-none-linux-gnu- pine64_plus_defconfig
```

**TODO** change extra settings ... see, embedded training lab instructions.

Finally, run the build.

``` bash
$ make CROSS_COMPILE=aarch64-none-linux-gnu-
```

### Create SD Card



``` bash
sudo dd if=./u-boot-sunxi-with-spl.bin of=/dev/sdb bs=1k seek=8
```


## Boot Environment


## Boot Scripts

To simplify multi level boot steps any sequence of u-boot commands may be stored in a file, stored on the device (or SD card), and executed from the command line or automatically.

A special script file is **boot.scr**; u-boot calls it automatically.

Well, kind of. It depends on the default environment settings. The **printenv** command shows the environment settings.
There is usually a variable, which determines which scripts may be executed, e.g.:

```
boot_scripts=boot.scr.uimg boot.scr
```

### mkimage

After chaning a boot script, you have to add a header for u-boot.

Use the **mkimage** command; the command line is:

```
$ mkimage -A arm -O linux -T script -C none -n "U-Boot boot script" -d boot.txt boot.scr
```

Transfer **boot.scr** to the first partition on the SD card (it is usually named **boot**).

### Boot From SD Card

This is the default setting for most bootable images.

The most generic settings are:

```
setenv bootargs console=ttyS0,115200 earlyprintk root=/dev/mmcblk0p2 rootwait

fatload mmc 0 $kernel_addr_r Image
fatload mmc 0 $fdt_addr_r sun50i-a64-pine64-plus.dtb

booti $kernel_addr_r - $fdt_addr_r
```

Please note, it does not use an initial RAM disk.

See also **boot.mmc.txt** for a more detailed version which can load an initial  RAM disk.
It verifies each step and stops execution on failure.

### Boot from TFTP Server

U-boot is able to boot from a TFTP server.

It loads the various files from the server to a specific RAM address. Any further steps are similar to booting from SD card.

If loading from the server fails, u-boot will enter the command prompt.

For a complete setup install a dhcp and a tftp server on your development PC, e.g.:

* tftpd-hpa - TFTP server
* isc-dhcp-server - DHCP server

We assume the DHCP server is running on **192.168.1.5** and serves addresses in that sub net.

The TFTP server is running on **192.168.1.5** and all the files are stored in the **a64** directory of the server.

Files involved:

* kernel: a64/Image
* device tree: a64/sun50i-a64-pine64-plus.dtb
* initial RAM disk: a64/initramfs-linux.img

The NFS server is running on **192.168.1.5** and _TBD_

The NFS server provides the full root file system of the device.

Please note, all the component may be derived from a distribution image or built with **buildroot**.

Available configurations:

* Kernel and Device Tree: **boot.net.txt**
* Kernel, Device Tree, and Initial RAM Disk: **boot.net.ramfs.txt**
* Kernel, Device Tree, and Root Filesystem vie NFS: **boot.net.nfs.txt**

## Setup SD Card

### Card Layout

[Sunxi Wiki: Bootable SD Card](http://linux-sunxi.org/Bootable_SD_card)

[U-Boot Sunxi](https://github.com/linux-sunxi/u-boot-sunxi/wiki)

The above pages describe how to set up an SD card which may be booted by an Allwinner SoC/CPU.

The card layout is:

| Sector | Start | Size  | Comment                                            |
| -----: | ----: | ----: | :------------------------------------------------- |
|    0   |    0  |   8KB | Unused, available for partition table etc.         |
|   16   |    8  |  32KB | Initial SPL loader                                 |
|   80   |   40  | 504KB | u-boot  (sector 64 / 32KB for 2013.07 and earlier) |
| 1088   |  544  | 128KB | environment                                        |
| 1344   |  672  | 128KB | Falcon mode boot params                            |
| 1600   |  800  |   --  | Falcon mode kernel start                           |
| 2048   | 1024  |   --  | Free for partitions (higher if using Falcon boot)  |

### Create SD Card

The files used in this description were created in the **buildroot** environment. They will be copied to the SD card.

The easiest way to get a bootable SD card is to write **sdcard.img** to an SD card using **dd**; **buildroot** created the image for you.

Creating a bootable SD card manually involve the following steps:

- Blank the SD card :warning:

```
$ sudo dd if=/dev/zero of=/dev/sdb bs=1M count=1
```

- Write the SPL and u-boot binaries :nut_and_bolt:

```
$ sudo dd if=sunxi-spl.bin of=/dev/sdb bs=1024 seek=8
$ sudo dd if=u-boot.bin of=/dev/sdb bs=1024 seek=40
```

- Create boot partition :boot:

Create a primary parition of about 100MB; it must be the first partition on the card.

Create a **vfat** filesystem.

See the **Boot Scripts** section above.

Copy a **boot.scr** file to the boot partition.

Depending on the configuration, you have to copy a kernel image, a device tree blob, and possibly an initial RAM file system to the boot partition and adjust the paths and file names in the script accordingly.

- Create root partition :house:

The root file system may be stored on the second partition; e.g. by extracting **rootfs.tar**.
