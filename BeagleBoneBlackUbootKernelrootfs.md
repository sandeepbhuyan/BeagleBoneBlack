# Building kernel-4.4 and Uboot-2018 and Ubuntu-16.04 Rootfs from scratch for the BeagleBoneBlack

<img src="BBBpicture/0.jpg" width="500">
![](BBBpicture/0.jpg)

## Introduction

Always i was getting `KERNEL PANIC`  :rage: or `Booting error` :weary: in my own BeagleBoneBlack Board.Here's a really "simple" guide on how to build U-Boot-2018 and Linux kernel-4.4 from scratch for the BeagleBone SoC. This is mostly a reminder of the steps for myself and shows what I recently learned from lots of different sources and by trial and error method.

This post describes the setup detail for installing Ubuntu based distro in BeagleBoneBlack Board. Details are described on:


- [x] Take board(BBB), Setting the host(X86 Intel Ubuntu-14.04).
- [x] Download and compile uboot`(MLO ,u-boot.img)` , dtb`(am335x-boneblack.dtb)` and and the Kernel`(zImage)` image on your board.
- [x] Install rootfs and Installing needed packages.
- [x] Setting with SD image.
- [x] Setting Ubuntu on target.

##  1. Download the GCC Linaro ARM Cross Compiler Toolchain:and extract it on Host machine(x86):
```
sandeeptux@sandeeplinux:~$ wget -c https://releases.linaro.org/components/toolchain/binaries/6.4-2017.11/arm-linux-gnueabihf/gcc-linaro-6.4.1-2017.11-x86_64_arm-linux-gnueabihf.tar.xz

sandeeptux@sandeeplinux:~$ tar xf gcc-linaro-6.4.1-2017.11-x86_64_arm-linux-gnueabihf.tar.xz  
```
**Create general variable environments:**
```
sandeeptux@sandeeplinux:~$ export ARCH=arm

sandeeptux@sandeeplinux:~$ export CROSS_COMPILE=/home/sandeeptux/gcc-linaro-6.4.1-2017.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
```

## Required Tools for Host machine (x86 intel Ubuntu-14.04) :

**_Note:Connect the USB side of the TTL cable to your computer.
Connect the wires to the J1 headers on your BeagleBone Black as shown:_**

Black wire to Pin 1

Green wire to Pin 4

White wire to Pin 5

![](BBBpicture/4.png)

I'm using here GTKterm as an example but any other debug terminal program can be used like [Minicom](http://processors.wiki.ti.com/index.php/Setting_up_Minicom_in_Ubuntu).


### 1-install minicom (command line serial port terminal)

`sandeeptux@sandeeplinux:~$ sudo apt-get install minicom`

`  sandeeptux@sandeeplinux:~$sudo minicom -s`
```


  	    +-----[configuration]------+
            | Filenames and paths      |
            | File transfer protocols  |
            | Serial port setup        |
            | Modem and dialing        |
            | Screen and keyboard      |
            | Save setup as dfl        |
            | Save setup as..          |
            | Exit                     |
            | Exit from Minicom        |
            +--------------------------+
then come down by using down arrow choose ---->Serial port setup
    +-----------------------------------------------------------------------+
    | A -    Serial Device      : /dev/ttyUSB0                              |
    | B - Lockfile Location     : /var/lock                                 |
    | C -   Callin Program      :                                           |
    | D -  Callout Program      :                                           |
    | E -    Bps/Par/Bits       : 115200 8N1                                |
    | F - Hardware Flow Control : No                                        |
    | G - Software Flow Control : No                                        |
    |                                                                       |
    |    Change which setting?                                              |
    +-----------------------------------------------------------------------+
    
``` 
## OR 
2. **_GTKterm - (GUI based serial port terminal)_**

Setup the communication with a terminal window so that commands can be exchanged with the application running in the Board on the development board.

`sandeeptux@sandeeplinux:~$ sudo apt-get update`

`sandeeptux@sandeeplinux:~$ sudo apt-get install gtkterm`


`sandeeptux@sandeeplinux:~$ dmesg | tail -8`
```

[19543.479877] hid-generic 0003:0E8F:0022.0005: input,hidraw2: USB HID v1.10 Device [GASIA USB KB V11] on usb-0000:00:14.0-2.4/input1
[19545.727769] usb 2-2.1: new full-speed USB device number 8 using xhci_hcd
[19545.816485] usb 2-2.1: New USB device found, idVendor=067b, idProduct=2303
[19545.816491] usb 2-2.1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[19545.816495] usb 2-2.1: Product: USB-Serial Controller
[19545.816498] usb 2-2.1: Manufacturer: Prolific Technology Inc.
[19545.817235] pl2303 2-2.1:1.0: pl2303 converter detected
[19545.818262] usb 2-2.1: pl2303 converter now attached to ttyUSB0

```
`sandeeptux@sandeeplinux:~$ sudo gtkterm`


* Provide your sudo password and the serial terminal GUI will appear. Click 'Configuration' and select 'Port', as shown in the figure below.
<img src="BBBpicture/1.png" width="500">
![](BBBpicture/1.png)

* In the pop-up window browse to the USB TTY port as shown in the figure below.
<img src="BBBpicture/2.png" width="500">
![](BBBpicture/2.png)

![](BBBpicture/3.png)

2.**_ncurses - (GUI based user interfaces in a terminal-independent manner)_** [ncurses](https://www.gnu.org/software/ncurses/ncurses.html).

```
sandeeptux@sandeeplinux:~$ sudo apt-get update
sandeeptux@sandeeplinux:~$ sudo apt install libncurses5-dev libncursesw5-dev

```
![](BBBpicture/5.png)


## Bootloader: U-Boot

The first step is to build the bootloader, U-Boot. Two important files will come out of this process: `MLO` and `u-boot.img`. MLO is what the U-Boot community calls the SPL (Secondary Program Loader) and contains executable code for the second boot stage. 
1. The on-board ROM initializes a 1 CPU cores will initialize and searches for a file called MLO on the first FAT partition of the first SD card, then loads it in memory and executes it
2. The SPL (MLO) initializes a few other things and searches on the same partition for u-boot.img, which is the third boot stage (the actual complete U-Boot program), loads it in memory and executes it
    
```
sandeeptux@sandeeplinux:~$ git clone https://github.com/u-boot/u-boot
sandeeptux@sandeeplinux:~$ cd u-boot/
sandeeptux@sandeeplinux:~$ git checkout v2018.01 
sandeeptux@sandeeplinux:~$ make am335x_boneblack_defconfig 
sandeeptux@sandeeplinux:~/u-boot$ ls
```
output file:

- [x] u-boot.bin-> is the binary compiled U-Boot bootloader.

- [x] u-boot.img-> contains u-boot.bin along with an additional header to be used by the boot ROM to determine how and where to load and execute U-Boot.

- [x] MLO-> The second stage bootloader is known as the SPL, but is sometimes referred to as the MLO.
            The SPL is the first stage of U-boot, and must be loaded from one of the boot sources into internal RAM.
            For MLO use the spl/u-boot-spl.bin file. The difference between u-boot-spl.bin and 
            MLO is that u-boot-spl.bin does not contain header information. Peripheral boot needs an MLO without header

Best Doc for TI Booting Process-> [TI The Boot Process](http://processors.wiki.ti.com/index.php/The_Boot_Process)

## Linux Kernel 

```
sandeeptux@sandeeplinux:~$ git clone https://github.com/beagleboard/linux.git
sandeeptux@sandeeplinux:~$ cd linux
sandeeptux@sandeeplinux:~/linux$ make bb.org_defconfig   
sandeeptux@sandeeplinux:~/linux$ make uImage LOADADDR=0x80008000 -j4
sandeeptux@sandeeplinux:~/linux$ make dtbs
sandeeptux@sandeeplinux:~/linux$ make modules -j4 
```
## OR

`sandeeptux@sandeeplinux:~/linux$ make uImage LOADADDR=0x80008000 -j4 uImage dtbs`

### Note: General error During Compilation 

### 1.Kernel compression Error 
```
sandeeptux@sandeeplinux:~/linux$ make uImage LOADADDR=0x80008000 -j4
LZO     arch/arm/boot/compressed/piggy.lzo
/bin/sh: 1: lzop: not found
make[2]: *** [arch/arm/boot/compressed/piggy.lzo] Error 1
make[1]: *** [arch/arm/boot/compressed/vmlinux] Error 2
make: *** [zImage] Error 2
```

### Error Solution

`sandeeptux@sandeeplinux:~/linux$ make menuconfig`
```
CONFIG_KERNEL_GZIP:                                                                                                                                                                      
The old and tried gzip compression. It provides a good balance between compression ratio and decompression speed.                                                                                                                                       

Symbol: KERNEL_GZIP 

 Location:
->General Setup
   -> Kernel compression mode  (<choice> [=y]) 
```
```

|─────────────────── Kernel compression mode ───────────────────┐
│  Use the arrow keys to navigate this window or press the      │  
│  hotkey of the item you wish to select followed by the <SPACE │ 
│  BAR>. Press <?> for additional information about this        │  
│ ┌───────────────────────────────────────────────────────────┐ │  
│ │                         (X) Gzip                          │ │  
│ │                         ( ) LZMA                          │ │  
│ │                         ( ) XZ                            │ │  
│ │                         ( ) LZO                           │ │  
│ │                         ( ) LZ4                           │ │  
│ │                                                           │ │ 
└─────────────────────────────────────────────────────────────  │  
├───────────────────────────────────────────────────────────────┤  
│                    <Select>      < Help >                     │  
└───────────────────────────────────────────────────────────────|
```
### 2.Kernel compression Error 

`Makefile:686: Cannot use CONFIG_CC_STACKPROTECTOR_STRONG: -fstack-protector-strong not supported by compiler`

### Error Solution 

` sandeeptux@sandeeplinux:~/linux$ export CROSS_COMPILE=/home/sandeeptux/gcc-linaro-6.4.1-2017.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-`

Let me explain what are the important files in `arch/arm/boot` those two targets will create while this is compiling:

- [x] `zImage`-> this is a self-extracting compressed kernel. You execute this file, and it has a decompression (gzip) algorithm to extract the rest of it, which is the actual kernel to execute.`zimage` can load any where in RAM.`zimage` is called position independent.
- [x] `uImage`-> this is `zImage` with a 64-byte U-Boot header so that U-Boot knows a few things when asked to boot this file.
                while `uImage` can load the  address has to provide.
```              
Ex: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage LOADADDR=0x80008000 -j4

LOADADDR :
                Start_address (hex)        End_address (hex)       Size            Description
  SDRAM          0x8000_0000               0xBFFF_FFFF             1GB             8-/16-bit External Memory
  
  uImage = Load address + absolute entry point 
```

- [x] `dts/am335x-bone.dtb`-> the compiled device tree "blob" which the kernel must have in order to initialize its drivers and SoC-specific routines`


## Creating a root filesystem

The kernel will be able to start booting with the above files, but won't be able to finish because it will complain it cannot find any root filesystem.

DOWNLOAD [Prebuilt Ubuntu-16 Rootfs](https://github.com/sandeepbhuyan/BeagleBoneBlackRootfsUbuntu-16.04/archive/master.zip)

### OR

`sandeeptux@sandeeplinux:~$ git clone https://github.com/sandeepbhuyan/BeagleBoneBlackRootfsUbuntu-16.04.git`

```
sandeeptux@sandeeplinux:~$ cd BeagleBoneBlackRootfsUbuntu-16.04/
sandeeptux@sandeeplinux:~/BeagleBoneBlackRootfsUbuntu-16.04$ ls
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
## `uEnv.txt` automating the booting process (for SD Card)
```
loadaddr=0x82000000
fdtaddr=0x88000000
rdaddr=0x88080000

initrd_high=0xffffffff
fdt_high=0xffffffff

#for single partitions:

mmcroot=/dev/mmcblk0p1
loadximage=load mmc 0:1 ${loadaddr} /boot/vmlinuz-${uname_r}
loadxfdt=load mmc 0:1 ${fdtaddr} /boot/dtbs/${uname_r}/${fdtfile}
loadxrd=load mmc 0:1 ${rdaddr} /boot/initrd.img-${uname_r}; setenv rdsize ${filesize}
loaduEnvtxt=load mmc 0:1 ${loadaddr} /boot/uEnv.txt ; env import -t ${loadaddr} ${filesize};
loadall=run loaduEnvtxt; run loadximage; run loadxfdt;

mmcargs=setenv bootargs console=tty0 console=${console} ${optargs} ${cape_disable} ${cape_enable} root=${mmcroot} rootfstype=${mmcrootfstype} ${cmdline}

uenvcmd=run loadall; run mmcargs; bootz ${loadaddr} - ${fdtaddr};
```

# SD Card


For SD Card if we have microSD ADAPTER ->> /dev/mmcblk0p1 ,/dev/mmcblk0p2 
```
sandeeptux@sandeeplinux:~$ dmesg | tail -3
[54971.955024]  mmcblk0: p1 p2
[54972.249471] FAT-fs (mmcblk0p1): Volume was not properly unmounted. Some data may be corrupt. Please run fsck.
[54972.255273] EXT4-fs (mmcblk0p2): mounted filesystem with ordered data mode. Opts: (null)
 ```
 
if we have SD Card USB ADAPTER ->> /dev/sdb1, /dev/sdb2,
```
sandeeptux@sandeeplinux:~$ dmesg | tail -4
[55342.206940]  sdb: sdb1 sdb2
[55342.688420] FAT-fs (sdb1): Volume was not properly unmounted. Some data may be corrupt. Please run fsck.
[55342.699352] EXT4-fs (sdb2): mounted filesystem with ordered data mode. Opts: (null)
[55345.084561] systemd-hostnamed[22617]: Warning: nss-myhostname is not installed. Changing the local hostname might make it unresolveable. Please install nss-myhostname!

```
```
Create Partition layout:
sandeeptux@sandeeplinux:~$ sudo fdisk /dev/sdb

Command (m for help): p

Disk /dev/sdb: 15.9 GB, 15931539456 bytes
64 heads, 32 sectors/track, 15193 cylinders, total 31116288 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1   *        2048     2099199     1048576    c  W95 FAT32 (LBA)
/dev/sdb2         2099200    31116287    14508544   83  Linux
```
Format partition 1 as FAT by typing `(sudo mkfs.vfat /dev/sdb1 )`

name change to `BOOT`- `sudo mkfs.vfat -F 32 -n "BOOT" /dev/sdb1`

Format partition 2 as ext4 by typing `(sudo mkfs.ext4 /dev/sdb2 )`

name change to `ROOTFS`- `sudo mkfs.ext4 -L "ROOTFS" /dev/sdb2`


### Now Final  Bringing it all together:

Creat 2 empty directory `(ex:boothost)`  `(ex:rootfshost)` for Copy Ubuntu-16.04 Rootfs from Host machine to SD card(SD Card USB ADAPTER) by help up virtual mount Setup SD card. In my case i have created 2 Directory in home Directory 

`1-boothost  (For /media/sandeeptux/BOOT) `

`2-rootfshost (For media/sandeeptux/ROOTFS)`
```
                      filesystem mount               External mount media device       copy contain
boothost<------------>/dev/sdb1/<--------------------->/media/sandeeptux/BOOT<------>MLO , u-boot.img 
rootfshost<---------->/dev/sdb2/<--------------------->/media/sandeeptux/ROOTFS<---->rootfs,zImage,uEnv.txt,am335x-boneblack.dtb
```
```
sandeeptux@sandeeplinux:~$ sudo mount /dev/sdb1/ boothost
sandeeptux@sandeeplinux:~$ sudo mount /dev/sdb2/ rootfshost

```
```
For 1st partion U-boot(MLO,u-boot.img):

sandeeptux@sandeeplinux:~/u-boot$ sudo cp MLO u-boot.img /home/sandeeptux/boothost/

For 2nd partion Rootfs (zImage,uEnv.txt,dtb):

sandeeptux@sandeeplinux:~$ cd BeagleBoneBlackRootfsUbuntu-16.04/

sandeeptux@sandeeplinux:~/BeagleBoneBlackRootfsUbuntu-16.04$ cp * -R rootfshost

sandeeptux@sandeeplinux:~/BeagleBoneBlackRootfsUbuntu-16.04$ sync

sandeeptux@sandeeplinux:~/linux/arch/arm/boot$ sudo cp zImage /home/sandeeptux/rootfshost/boot

sandeeptux@sandeeplinux:~/linux/arch/arm/boot/dts$ sudo cp am335x-* /home/sandeeptux/rootfshost/boot

sandeeptux@sandeeplinux:~$ sudo cp uEnv.txt rootfshost/boot

sandeeptux@sandeeplinux:~$ sudo chown root:root rootfshost

sandeeptux@sandeeplinux:~$ sudo chmod 755 rootfshost

sandeeptux@sandeeplinux:~$ sudo chown root:root rootfshost/usr/bin/sudo  

sandeeptux@sandeeplinux:~$ sudo chmod 4755 rootfshost/usr/bin/sudo

sandeeptux@sandeeplinux:~$ sudo umount boothost

sandeeptux@sandeeplinux:~$ sudo umount rootfshost

```
 ## Note: If you have issues with sudo on user UID in BeagleboneBlack kernel ,
 
error : `sudo: /usr/bin/sudo must be owned by uid 0 and have the setuid bit set`
```
solution:
sandeeptux@sandeeplinux:~$ sudo chown root:root rootfshost//usr/bin/sudo  
sandeeptux@sandeeplinux:~$ sudo chmod 4755 rootfshost/usr/bin/sudo
```

## Edit networks interfaces and append in the existing file:
```
$ sudo nano etc/network/interfaces  
auto lo  
iface lo inet loopback  
auto eth0  
iface eth0 inet dhcp
```

Remove SD:

sandeeptux@sandeeplinux:~$ sudo umount /media/sandeeptux/BOOT
sandeeptux@sandeeplinux:~$ sudo umount /media/sandeeptux/ROOTFS
