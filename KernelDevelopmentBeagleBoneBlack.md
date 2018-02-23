### Linux Kernel Driver Programming with Embedded Devices

Note : Here i have taken the SOC called  `Beagle Bone Black(TI AM335X)` for Development (you can take any SOC)

1. Writing a Linux Kernel Module

- [x] Compile Kernel
```
sandeeptux@sandeeplinux:~$ git clone https://github.com/beagleboard/linux.git
sandeeptux@sandeeplinux:~$ cd linux
sandeeptux@sandeeplinux:~/linux$ make bb.org_defconfig   
sandeeptux@sandeeplinux:~/linux$ make uImage LOADADDR=0x80008000 -j4
sandeeptux@sandeeplinux:~/linux$ make dtbs
```
- [x] Compile Module

`sandeeptux@sandeeplinux:~/linux$ make modules -j4 `

- [x] Install Module

`sandeeptux@sandeeplinux:~/linux$ sudo make ARCH=arm CROSS_COMPILE=/home/sandeeptux/gcc-linaro-6.4.1-2017.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf- INSTALL_MOD_PATH=/media/sandeeptux/ROOTFS/ modules_install`



