Gentoo_GPTlvmSysD
=================
This is a `HowTo` step by step instruction of [Gentoo][gentoo] installation on [PC hardware][pc] or [Virtual Machine][virtualmachine] (VM). There is a nice official documentation [HERE][gentoobook], but the aim of this `HowTo` is short step by step command line procedure for future faster installation. This is my attempt to put it all together. The full explanation located onto official [Gentoo WEB site][gentoo].  

For [Virtual Machines][virtualmachine] (VM) the kernel configuration MUST be changed according to the [official Gentoo documentation][gentoobook] before kernel compilation.  

This `HowTo` is dedicated to [Gentoo][gentoo] installation with [LVM][lvm] and [SystemD][systemd]. `SystemD` is a modern `SysV`-style init and `rc` replacement for `Linux` systems. It is supported in [Gentoo][gentoo] as an alternative `init` system.  

Unfortunately **NO** exists official documentation regarding [Gentoo][gentoo] [SystemD][systemd] *step by step* installation. That is why I have to create it myself.  

The way of installation specific for me, may **NOT** work correctly and can be changed further.  

### Installation procedures  

First of all check out your [PC][pc] has [Ethernet][ethernet] connection to the Internet. The [Gentoo][gentoo] is downloadable by Internet and without it is **NOT** possible to install.  

To make partitions on [GPT][guid] partition disk I prefer to use command line [parted][parted] program as described in [official documentation][gentoobook].  

To install [Gentoo][gentoo] you can use official [minimal installation CD][gentoomini] which is recommended, but by my experience the better to use `third party` [System Rescue CD][sysrescuecd] which is more friendly for installation.  

The [System Rescue CD][sysrescuecd] is [Arch Linux][archlnx] based **live CD** with [SystemD][systemd] on board.  

#### Pre-installation:  

 1.  Boot the [System Rescue CD][sysrescuecd].  
 2.  Check if you have Internet connection by `ip a` and `ping google.com` or `ping 1.1.1.1` command. If **NOT** you have to solve this problem before installation.  
 3.  If you wish to connect to your installation by `SSH` from other computer you must change the password for `root` by `passwd` command and stop the `iptables` firewall which is running on this CD by `systemctl stop iptables`. Then connect to your PC by IP taken from `ip a` command.  
 4.  If you successfully connect to your system by `SSH` as `root` - you are on the way *to the next steps*.  
 5.  Check the current date with `date` command and set it by: `ntpd -q -g`. This is very important for correct `Gentoo` installation.  

#### General Installation procedure (SystemD LVM install on [GPT][guid]):  

 1.  Check you hard drive configuration by `lsblk -f` command.  
 Be careful and DO NOT format the partition with important data. **YOU WERE WARNED**!  
 Based on official documentation the standard partitioning scheme for [LVM][lvm] is:  
 ```
 /dev/sda1 (bootloader) 2M   BIOS boot partition
 /dev/sda2 ext2         128M Boot/EFI system partition
 /dev/sda3 LVM          MAX  LVM partition
 ```
 ( **NOTE**: If you plan to use the `SWAP` partition on your system better place it outside [LVM][lvm] cause I had problems with boot with [SystemD][systemd] enabled. )  

 2.  Run the: `# parted -a optimal /dev/sda` for `sda` hard drive (**name depends on your system**).  
 3.  Clean up the disk and make `GPT` label by: `(parted) mklabel gpt`.  
 4.  Setup the UNIT by: `(parted) unit mib`  
 5.  Add additional commands for [LVM][lvm] creation  and check result by `print` command. Then `quit` from there:  
 ```
 (parted) mkpart primary 1 3
 (parted) name 1 grub
 (parted) set 1 bios_grub on
 (parted) mkpart primary 3 131
 (parted) name 2 boot
 (parted) set 2 boot on
 (parted) mkpart primary 131 2179
 (parted) name 3 swap
 (parted) mkpart primary 2179 -1
 (parted) name 4 lvm01
 (parted) set 4 lvm on
 ```
 ( **NOTE**:  The `SWAP` partition size depends on your system `RAM`. Look [HERE][swapsize] for your reference. I will use `2Gb` here.)  

 6.  Physical volumes can be created / initialized with the `pvcreate` command:  
 ```
 # pvcreate /dev/sda4
 ```
 7.  With the `pvdisplay` command, an overview of all active physical volumes on the system can be obtained. If more physical volumes should be displayed, then `pvscan` can detect inactive physical volumes and activate those:  
 ```
 # pvdisplay
 # pvscan
 ```
 8.  Let's create the Volume Group (**VG**):  
 ```
 # vgcreate vg01 /dev/sda4
 ```
 9.  Let's create the Logical Volumes (**LV**) by:  
 ```
 # lvcreate -l 100%VG -n rootfs vg01
 ```
 10. Now lets format the created Logical Volumes (**LV**) by:  
 ```
 # mkfs.ext2 /dev/sda2
 # mkfs.ext4 /dev/vg01/rootfs
 ```
 ( **NOTE**: First for `BOOT` and the second for `ROOT` )  

 11. Activate the `SWAP` logical partition by:  
 ```
 # mkswap /dev/sda3
 # swapon /dev/sda3
 ```
 12. Now mount the `Root` by:  
 ```
 # mkdir /mnt/gentoo
 # mount /dev/vg01/rootfs /mnt/gentoo
 ```
 13. Go to the [Gentoo][gentoo] folder:  
 ```
 # cd /mnt/gentoo
 ```
 14. Download `Stage 3 systemd` tarball from [HERE][gentoodownload] by `curl` command:  
 ```
 # curl -LO https://(your tarball link here)
 ```
 15. Unpack the `Stage 3 systemd` tarball by:  
 ```
 # tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
 ```
 ( **NOTE**: You can remove now the downloaded archive by: `rm /stage3-*.tar.*` )  

 16. Configuring compile options by:  
 ```
 # vim /mnt/gentoo/etc/portage/make.conf
 ```
 Add the `-march=native` and `-j4` there ( good choice is the number of CPUs / CPU cores in the system for -j ):  
 ```
 COMMON_FLAGS="-march=native -O2 -pipe"
 MAKEOPTS="-j4"
 ```
 ( **NOTE**: Read [THIS][makeopts] article also. )  

 17. Cause `mirrorselect` is absent on `System Rescue CD` we will do it manually. Official information about mirrors are [HERE][gentoomirrors]. Add into `/mnt/gentoo/etc/portage/make.conf` below the mirrors specific for your country. For `RU` it will be:  
 ```
 GENTOO_MIRRORS="https://mirror.yandex.ru/gentoo-distfiles/ https://gentoo-mirror.alexxy.name/"
 ```
 18. Create the `ebuild` repository:  
 ```
 # mkdir -p /mnt/gentoo/etc/portage/repos.conf
 # cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
 # cat /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
 ```
 19. Copy `DNS` info by:  
 ```
 # cp -L /etc/resolv.conf /mnt/gentoo/etc/
 ```
 ( **NOTE**: If you abort the installation you can start over from this step! )  

 20. Mounting the necessary file systems by:  
 ```
 # mount --types proc /proc /mnt/gentoo/proc
 # mount --rbind /sys /mnt/gentoo/sys
 # mount --make-rslave /mnt/gentoo/sys
 # mount --rbind /dev /mnt/gentoo/dev
 # mount --make-rslave /mnt/gentoo/dev
 ```
 21. Entering the `chroot` environment by:  
 ```
 # chroot /mnt/gentoo /bin/bash
 # source /etc/profile
 # export PS1="(chroot) ${PS1}"
 ```
 ( **NOTE**: Also you can make the [BASH][bash] script in root folder for future [Gentoo][gentoo] if you do this frequent. )  
 ```bash
 #!/usr/bin/env bash

 mount --types proc /proc /mnt/gentoo/proc
 mount --rbind /sys /mnt/gentoo/sys
 mount --make-rslave /mnt/gentoo/sys
 mount --rbind /dev /mnt/gentoo/dev
 mount --make-rslave /mnt/gentoo/dev

 echo ''
 echo 'Run it after shell script execution:'
 echo 'source /etc/profile'
 echo 'export PS1="(chroot) ${PS1}"'

 chroot /mnt/gentoo /bin/bash
 ```
 22. **Don't forget** to mount the `boot` partition by:  
 ```
 # mount /dev/sda2 /boot
 ```
 23. Install the [Gentoo][gentoo] `ebuild` repository snapshot from the `WEB` by:  
 ```
 # emerge-webrsync
 ```
 24. Purge the news if you are not going to read them by:  
 ```
 # eselect news read all
 # eselect news purge
 ```
 25. Choosing the right profile. `eselect profile set (correct number)` We will use `default` one. Check it by:  
 ```
 # eselect profile list
 ```
 26. Update the @world set:  
 ```
 # emerge -avuDN @world
 ```
 ( **NOTE**: It may TAKE A LOT OF TIME depends on your hardware. *You can skip it now*. )  

 27. Set the `Timezone`. For `RU/Moscow` it will be:  
 ```
 # echo "Europe/Moscow" > /etc/timezone
 # emerge --config sys-libs/timezone-data
 ```
 28. Setup `VIM` cause I hate to use `Nano` by:  
 ```
 # emerge -a vim
 ```
 29. Now generate the locale. Add this into `/etc/locale.gen` for US ones:  
 ```
 cat <<EOF >> /etc/locale.gen
 en_US ISO-8859-1
 en_US.UTF-8 UTF-8
 EOF
 ```
 30. Run this command:  
 ```
 # locale-gen
 ```
 31. Make `UTF-8` locale selection by:  
 ```
 # eselect locale list
 # eselect locale set (correct number)
 ```
 32. Reload `envirionment`:  
 ```
 # env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
 ```
 33. Install `LVM2` and `SystemD` by:  
 ```
 # emerge -a sys-fs/lvm2 sys-apps/systemd
 ```
 34. Install the `Kernel` sources by:  
 ```
 # emerge -a sys-kernel/gentoo-sources
 # ls -l /usr/src/linux
 ```
 35. Create the configuration file for future `Kernel` compilation:  
 ```
 # cd /usr/src/linux
 # make defconfig
 # make menuconfig
 ```
 ( **NOTE**: For correct Kernel `LVM` and `SystemD` configuration follow the [LVM][gentoolvm] and [SystemD][systemd] official instructions. )  

 **Enabling LVM**  
 ```
 Device Drivers  --->
   Multiple devices driver support (RAID and LVM)  --->
       <*> Device mapper support
           <*> Crypt target support
           <*> Snapshot target
           <*> Mirror target
           <*> Multipath target
               <*> I/O Path Selector based on the number of in-flight I/Os
               <*> I/O Path Selector based on the service time
 ```
 **Quick setup SystemD using Gentoo-sources**  
 ```
 Gentoo Linux --->
   Support for init systems, system and service managers --->
      [*] systemd
  ```
 36. You can configure Kernel manually which is most effective way. Anyway, setup `genkernel` for future use. See below.  
 ```
 # emerge -a sys-kernel/genkernel
 # vim /etc/fstab
 ```
 Add this into `fstab` file. This based on default `LVM` configuration (pay attention to tabs between words):  
 ```
 /dev/sda2         /boot       ext2  defaults,noatime  0 2
 /dev/sda3         none        swap  sw                0 0
 /dev/vg01/rootfs  /           ext4  noatime           0 1
 ```
 37. Compile the `Kernel` by:  
 ```
 # make -j4 && make modules_install && make install
 ```
 ( **NOTE**: `-j4` for four CPU cores processor. )  

 38. Create `initramfs` by:  
 ```
 # genkernel --lvm --install initramfs
 ```
 ( **NOTE**: Check if `initramfs` was created by: `ls -la /boot`. )  

 39. Setup `Grub 2` boot loader ( for `sda` in our case ):  
 ```
 # echo 'sys-boot/grub:2 device-mapper' >> /etc/portage/package.use/package.use
 # emerge -av sys-boot/grub:2
 # grub-install /dev/sda
 ```
 ( **NOTE**: `Grub 2` should be compiled with device-mapper support. )  

 40. Change variables in the `Grub 2` configuration file `/etc/default/grub` by:  
 ```
 # vim /etc/default/grub
 ```
 ( **NOTE**: Un-comment `GRUB_CMDLINE_LINUX_DEFAULT="dolvm"` and `GRUB_CMDLINE_LINUX="init=/lib/systemd/systemd"`. )  

 41. Then generate the `Grub 2` configuration file by:  
 ```
 # grub-mkconfig -o /boot/grub/grub.cfg
 ```
 ( **NOTE**: Ignore the `WARNINGS: /run/lvm/lvmetad.socket: connect failed`. )  

 42. For network configuration create this file `/etc/systemd/network/50-dhcp.network` and add:  
 ```
 cat <<EOF > /etc/systemd/network/50-dhcp.network
 [Match]
 Name=en*

 [Network]
 DHCP=yes
 EOF
 ```
 ( **NOTE**: Correct the `Name` according to your network interface name. )  

 43. Install `sudo` by:  
 ```
 # emerge -av sudo
 ```
 44. Change password for **root**, create the **User** for your system and setup the **password** for him:  
  ```
 # passwd
 # useradd -m -G users,wheel,audio -s /bin/bash username
 # passwd username
 ```
 ( **NOTE**: Don't forget to unblock the wheel group in **SUDO** `config` file, check if user can access root account and lock the `root` account by: `sudo passwd -l root`. DO NOT login as `root` )  

 45. Let's exit the `chroot` environment and enter `systemd` name space by:  
 ```
 # exit
 # cd
 # umount -l /mnt/gentoo/dev{/shm,/pts,}
 # mount /dev/sda2 /mnt/gentoo/boot
 # systemd-nspawn -bD /mnt/gentoo
 ```
 46. If booted correctly setup locale, hostname and timezone ones again:  
 ```
 # systemd-machine-id-setup
 # localectl set-locale LANG=en_US.utf8
 # localectl set-keymap us
 # hostnamectl set-hostname gentoobox
 # timedatectl set-timezone Europe/Moscow
 # timedatectl set-ntp yes
 ```
 47. Add into `/etc/hosts`:  
 ```
 # This defines the current system and must be set
 127.0.0.1 gentoobox.localnetwork gentoobox localhost
 ```
 48. Enable the network service by:  
 ```
 # systemctl enable systemd-networkd.service
 ```
 49. Enable the `LVM2` service by:  
 ```
 # systemctl enable lvm2-monitor.service
 ```
 50. Now lets leave `systemd` name space by **poweroff** command:  
 ```
 # poweroff
 ```
 51. Setup almost complete. Reboot now:  
 ```
 # cd
 # umount -R /mnt/gentoo
 # reboot
 ```
 52. If reboot success you have installed `Gentoo` as `SystemD` on `LVM2`. Congratulations.  

### License  

This text has been written by ©2020 DimiG

[gentoo]:https://www.gentoo.org/
[pc]:https://en.wikipedia.org/wiki/Personal_computer
[virtualmachine]:https://en.wikipedia.org/wiki/Virtual_machine
[gentoobook]:https://wiki.gentoo.org/wiki/Handbook:AMD64
[lvm]:https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)
[systemd]:https://wiki.gentoo.org/wiki/Systemd
[ethernet]:https://en.wikipedia.org/wiki/Ethernet
[guid]:https://en.wikipedia.org/wiki/GUID_Partition_Table
[parted]:https://en.wikipedia.org/wiki/GNU_Parted
[gentoomini]:https://www.gentoo.org/downloads/
[sysrescuecd]:https://www.system-rescue-cd.org/
[archlnx]:https://www.archlinux.org/
[swapsize]:https://itsfoss.com/swap-size/
[gentoodownload]:https://gentoo.org/downloads/
[gentoomirrors]:https://www.gentoo.org/downloads/mirrors/
[makeopts]:https://blogs.gentoo.org/ago/2013/01/14/makeopts-jcore-1-is-not-the-best-optimization/
[bash]:https://en.wikipedia.org/wiki/Bash_(Unix_shell)
[gentoolvm]:https://wiki.gentoo.org/wiki/LVM
