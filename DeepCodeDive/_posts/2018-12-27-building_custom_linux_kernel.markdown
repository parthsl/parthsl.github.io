---
layout: post
title:  "Building Custom Linux Kernel!"
date:   2018-12-27 20:06:36 +0530
categories: kernel
---

Linux kernel is a nice piece of collaborative work developed by thousands of developers across the world. Hence it always feels proud to get our hands dirty with such art. 

This post gives a begineer's guide to how to compile the kernel in your system for the first time and install it.

#### Get the kernel sources
```sh
git clone https://www.github.com/torvalds/linux.git
```

#### Build the dependencies
Install packages for compiler, curses, flex, bison, etc based on your linux distro
- For `dnf/yum` based distro
```sh
sudo dnf install bison automake autoconf flex ncurses-devel elfutils-libelf-devel make gcc elfutils-devel libunwind-devel xz-devel numactl-devel openssl-devel binutils-devel
```
- For `apt` based distro
```sh
sudo apt install flex libncurses5 elfutils openssl libssl-dev automake bison pkg-config libncurses5-dev
```

#### Configure kernel
Generally linux kernel has thousands of parameters and modules to be configured based on the system architecture. 

Linux kernel  maintains `.config` file in root folder of kernel source tree. One can use any editor to configure the file but the curses based builtin tool makes it easy to change any configurations.

For begginer's one can get default config file for x86\_64 platform by below command, while for other architectures like PowerPC,MIPS,arm64,etc. look thorugh `arch/<architecture name>/configs/` folder to find proper name.
```sh
make x86_64_defconfig
```
To traverse through the config file, just use `make menuconfig` command to fire up curses based window for easier navigation.

#### Compile kernel
Compiling of kernel goes through compilation of several files which may take time in several minutes to hours in typical desktop systems. The make with `-j<nr_threads>` arguments helps out here by parallel building. `nproc` total CPUs in the system here.
```sh
make -j`nproc`
```

Once compiled succesfully, the output is a `vmlinux` file in the root directory which is basically the kernel. 

#### Installing kernel
- Kernel built vmlinux is the file which gets executed firstly after system boot, hence it cannot depend on any other code or library as it itself has the loader and linker. For this reason, the vmlinux file is statically build. The kernel installation gets done in three parts: installation of vmlinux, kernel modules and initrd image.

- The vmlinux needs to copied at `/boot` partition along with its `System.map` and `.config` file as optional things.

- The `vmlinux` contains module codes, which means it has capability to add some code to itself after it gets executed. This modules are basically drivers, energy governors, etc.  which needs to be added along ith the kernel. This can be invoked by `make modules_install` which will compile relevant modules configured using `.config` file and install it to `/lib/modules/` directory.

- The system boot sequence is as follows:
	1. The power on button invokes the system firmware/BIOS which in turn powers on motherboard components including disks.
	2. The control is then passed on to the boot loader like GRUB/LILO seating in /boot partition of the disk. It is the one which provides options to select a kernel to be loaded.
	3. The boot loader then invokes the initrd image which does mounting of several disk paritions like `/root`, `/home`/, `/swap`, etc. and boots vmlinux from there in.
The initrd thus has to be generated using `mkinitrd` command and boot loader has to be updated for the new kernel addition.

- The Makefile does all the above three steps and updates the boot loader using below command
```sh
sudo make install
```

- **Manual method**
```sh
make modules_install -j`nproc`
```
Get kernel-version-name from `/include/config/kernel.release` file.
```sh
mkinitramfs -o initramfs-<kernel-version-name>.img <kernel-version-name>
cp initramfs--<kernel-version-name>.img /boot
cp .config /boot/config--<kernel-version-name>
cp System.map /boot/System.map--<kernel-version-name>
cp vmlinux /boot/vmlinuz--<kernel-version-name>
```

#### Booting new kernel
- If succesfully installed the new kernel, then one can reboot the system and find the kernel with its version installed in boot loader. Just select that kernel to start the system with your new kernel.
After system start one can use `uname -a` to find the current kernel info.
- Other way to start the new kernel is to build custom initrd from above manual steps and invoke below command
```sh
kexec ./vmlinux "./initrd.img" --append="`cat /proc/cmdline`"
```
The advantage of above kexec command is that it will load the new kernel without rebooting the system. Also even with just modul installation step only one can test this kernel without updating boot loader or `/boot` partition.


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
