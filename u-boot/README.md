# U-Boot

**Das U-Boot** (subtitled "the Universal Boot Loader" and often shortened to U-Boot) is an open source, primary boot loader used in embedded devices to package the instructions to boot the device's operating system kernel. It is available for a number of computer architectures, including 68k, ARM, Blackfin, MicroBlaze, MIPS, Nios, SuperH, PPC, RISC-V and x86.


## Boot Environment



## Boot Scripts

To simplify multi level boot steps any sequence of u-boot commands may be stored in a file, stored on the device, and executed from the command line or automatically.

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

It loads the various file from the server to a specific RAM address. Any further steps are similar to booting from SD card.

If loading from the server fails, u-boot will enter the command prompt.

For a complete setup install a dhcp and a tftp server on your development PC:

* tftpd-hpa - TFTP server
* isc-dhcp-server - DHCP server

We assume the DHCP server is running on **192.168.1.5** and serves addresses in that sub net.

The TFTP server is running on **192.168.1.5** and all the files are stored in the **a64** directory of the server. 

Files involved:

* kernel: a64/Image
* device tree: a64/sun50i-a64-pine64-plus.dtb
* initial RAF disk: a64/initramfs-linux.img

The NFS is running on **192.168.1.5** and _TBD_

The NFS server provides the full root fle system of the device.

Please note, all the component may be derived from a distribution image or built with **buildroot**.

* Kernel and Device Tree: **boot.net.txt**
* Kernel, Device Tree, and Initial RAM Disk: **boot.net.ramfs.txt**
* Kernel, Device Tree, and Root Filesystem vie NFS: **boot.net.nfs.txt**


