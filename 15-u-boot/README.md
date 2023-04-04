# Das U-Boot

**Das U-Boot** (subtitled "the Universal Boot Loader" and often shortened to U-Boot) is an open source, primary boot loader used in embedded devices to package the instructions to boot the device's operating system kernel. It is available for a number of computer architectures, including 68k, ARM, Blackfin, MicroBlaze, MIPS, Nios, SuperH, PPC, RISC-V and x86.


## Build U-Boot From Scratch

The **[UBOOT]** distribution comes with a lot of documetation.

The build steps for the Pine 64 board are described in ***./board/sunxi/README.sunxi64***.


Before building U-Boot, you have to build two other packages.


### Build SCP Firmware - Crust

SCP firmware is responsible for implementing system suspend/resume, and (on
boards without a PMIC) soft poweroff/on.

It is an optional component; you may skip this step.

As **Crust** runs on a different architechture, you need a different too chain for for building the software.

See **[OPNRSC]** for toolcahin binaries.

Export the following environment variables.

``` bash
 ARCH         : or1k
 CROSS_COMPILE: or1k-linux-musl-
 SYSROOT      : /home/md/opt/x-tools/or1k-linux-musl-cross/or1k-linux-musl
 PATH         : /home/md/opt/x-tools/or1k-linux-musl-cross/bin
```

Download the SCP firmware source code from **[CRUST]**

``` bash
$ git clone https://github.com/crust-firmware/crust
$ cd crust
```

Select the configuration for Pine 64 and build the binaries:

``` bash
$ make pine64_plus_defconfig
$ make scp
```
You should find **scp.bin** in *./build/scp/scp.bin*.

Make it available for the u-boot build:

``` bash
$ export SCP=$(pwd)/build/scp/scp.bin
```

If you do not want to use SCP firmware, set **SCP** to ***/dev/null***.


### Build ARM Trusted Firmware

Trusted Firmware-A (TF-A) is a reference implementation of secure world software for Arm A-Profile
architectures (Armv8-A and Armv7-A), including an Exception Level 3 (EL3) Secure Monitor.

Download the Trusted Firmware sources from **[ATFS1]** or **[ATFS2]**.

``` bash
$ git clone https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git
$ cd trusted-firmware-a/
$ git checkout master
```

There are build instructions in ***./docs/plat/allwinner.rst***.

Select the default configuration for the board:

``` bash
$ make pine64_plus_defconfig
```

See ***allwinner.rst*** for possible customizations.

Finally, run the build; the target **bl31** should be sufficient.

``` bash
$ make CROSS_COMPILE=aarch64-none-linux-gnu- DEBUG=1 PLAT=sun50i_a64
```

You should find **bl31.bin** in *./build/sun50i_a64/debug* or *./build/sun50i_a64/release*.

Make it available for the u-boot build:

``` bash
$ export BL31=$(pwd)/build/sun50i_a64/debug/bl31.bin
```


## Build U-Boot

Finally build the U-Boot binary which includes TF-A and Crust.

Clone the U-Boot sources from **[UBOOT]**

``` bash
$ git clone https://gitlab.denx.de/u-boot/u-boot
$ cd u-boot
$ git checkout v2022.07
```

You may have to check out a different stable version.

There are build instructions in **./board/sunxi/README.sunxi64** and **./doc/board/allwinner/sunxi.rst**.

Specify the paths to the **BL31** and **SCP** binary files; see steps above.

Select the default configuration for the board and customize some settings:

``` bash
$ make CROSS_COMPILE=aarch64-none-linux-gnu- pine64_plus_defconfig
$ make CROSS_COMPILE=aarch64-none-linux-gnu- menuconfig
```

Change the following settings:

  * **Environment** - disable **Environment is not stored**
  * **Environment** - enable **Environment is in a EXT4 filesystem**, disble all other options for storin the  environment (MMC,SPI, ...)
  * **Environment** - **Name of block device for the environment:** - **mmc**
  * **Environment** - **Device and partition for where to store the environment in EXT4:** - **0:4**
  * **Environment** - **Name of the EXT4 file to use for the environment:** - **/uboot.env**
  * **Device Drivers - Watchdog Timer Support** - disable **IWDG watchdog ...** *obsolete??*

Finally, run the build.

``` bash
$ make CROSS_COMPILE=aarch64-none-linux-gnu- DEVICE_TREE=sun50i-a64-pine64-plus
```


## Create SD Card

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


### Flash U-Boot

``` bash
sudo dd if=./u-boot-sunxi-with-spl.bin of=/dev/sdb bs=1k seek=8
```

### Partitions

**TODO**


## Boot Environment

Once you have a bootable SD card, there are several options to boot the Linux kernel and system.


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


### Boot From SD Card

This is the default setting for most bootable images.

The most generic settings are:

```
setenv bootargs console=ttyS0,115200 earlyprintk root=/dev/mmcblk0p2 rootwait ## TODO: verify partition; see above

fatload mmc 0 $kernel_addr_r Image
fatload mmc 0 $fdt_addr_r sun50i-a64-pine64-plus.dtb

booti $kernel_addr_r - $fdt_addr_r
```

Please note, it does not use an initial RAM disk.

See also **boot.mmc.txt** for a more detailed version which can load an initial  RAM disk.
It verifies each step and stops execution on failure.



## Boot Scripts

To simplify multi level boot steps any sequence of u-boot commands may be stored in a file,
stored on the device (or SD card), and executed from the command line or automatically.

A special script file is **boot.scr**; u-boot calls it automatically.

Well, kind of. It depends on the default environment settings.

The **printenv** command shows the environment settings.
There is usually a variable, which determines which scripts will be executed, e.g.:

```
boot_scripts=boot.scr.uimg boot.scr
```

### mkimage

After changing any boot script, you have to add a header for u-boot.

Use the **mkimage** command; the command line is:

```
$ mkimage -A arm -O linux -T script -C none -n "U-Boot boot script" -d boot.txt boot.scr
```

Transfer **boot.scr** to the first partition on the SD card (it is usually named **boot**).






### Create SD Card (-> move to buildroot chapter)

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



# Links
## Sources
* **[UBOOT]** ["Das U-Boot" Source Tree](https://source.denx.de/u-boot/u-boot)arm trusted firmware

* **[UBOOT]** ["Das U-Boot" GitHub Mirror](https://github.com/u-boot/u-boot)

* **[ATF]** [ARM Trusted Firmware (ATF)](https://www.trustedfirmware.org/)

* **[ATFS1]** [Trusted Firmware A (Git)](https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git)

* **[ATFS2]** [Trusted Firmware A (Git Mirror)](https://github.com/ARM-software/arm-trusted-firmware)

* **[CRUST]** [Libre SCP firmware for Allwinner sunxi SoCs](https://github.com/crust-firmware/crust)
