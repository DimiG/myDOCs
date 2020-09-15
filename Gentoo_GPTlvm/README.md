Gentoo_GPTlvm
=============
This is a `HowTo` step by step instruction of [Gentoo][gentoo] installation on [PC hardware][pc] or [Virtual Machine][virtualmachine] (VM). There is a nice official documentation [HERE][gentoobook], but the aim of this `HowTo` is short step by step command line procedure for future faster installation. This is my attempt to put it all together. The full explanation located onto official [Gentoo WEB site][gentoo].  

For [Virtual Machines][virtualmachine] (VM) the kernel configuration MUST be changed according to the [official Gentoo documentation][gentoobook] before kernel compilation.  

The way of installation specific for me and can be changed further.  

### Installation procedures  

First of all check out your [PC][pc] has [Ethernet][ethernet] connection to the Internet. The [Gentoo][gentoo] is downloadable by Internet and without it is NOT possible to install.  

To make partitions on [GPT][guid] partition disk I prefer to use command line [parted][parted] program as described in [official documentation][gentoobook].  

To install [Gentoo][gentoo] you can use official [minimal installation CD][gentoomini] which is recommended, but by my experience the better to use `third party` [System Rescue Cd][sysrescuecd] which is more friendly for installation.  

The [systemd][systemd] variant of [Gentoo][gentoo] is NOT possible to install at all with original minimal installation CD. And there is not complete description how to do it in official documentation.  

#### Pre-installation:  

 1.  Boot the [System Rescue CD][sysrescuecd].  
 2.  Check if you have Internet connection by `ip a` and `ping google.com` or `ping 1.1.1.1` command. If NOT you have to solve this problem before installation.  
 3.  If you wish to connect to your installation by `SSH` from other computer you must change the password for `root` by `passwd` command and stop the `iptables` firewall which is running on this CD by `systemctl stop iptables`. Then connect to your PC by IP from `ip a` command.  
 4.  If you successfully connect to your system by `SSH` as `root` - you are on the way to the next steps.  
 5.  Check the current date with `date` command and set it by: `ntpd -q -g`. This is very important for correct `Gentoo` installation.  

#### General Installation procedure (LVM install on [GPT][guid]):  

 1.  Check you hard drive configuration by `lsblk -f` command.  
 Be careful and DON'T format the partition with important data. **YOU WERE WARNED**!  
 Based on official documentation the standard partitioning scheme for [LVM][lvm] is:  
 ```
 /dev/sda1 (bootloader) 2M   BIOS boot partition
 /dev/sda2 ext2         128M Boot/EFI system partition
 /dev/sda3 LVM          MAX  LVM partition
 ```
 2.  Run the: `# parted -a optimal /dev/sda` for `sda` hard drive (name depends on your system).  
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
 (parted) mkpart primary 131 -1
 (parted) name 3 lvm01
 (parted) set 3 lvm on
 ```
 6.  Physical volumes can be created / initialized with the pvcreate command:  
 ```
 # pvcreate /dev/sda3
 ```
 7.  With the `pvdisplay` command, an overview of all active physical volumes on the system can be obtained. If more physical volumes should be displayed, then `pvscan` can detect inactive physical volumes and activate those:  
 ```
 # pvdisplay
 # pvscan
 ```
 8.  Let's create the Volume Group (VG):  
 ```
 # vgcreate vg01 /dev/sda3
 ```
 9.  Let's create the Logical Volumes (LV) by:  
 ```
 # lvcreate -L 512M -n swap vg01
 # lvcreate -l 100%VG -n rootfs vg01
 ```
 10. Now lets format the created Logical Volumes (LV) by:  
 ```
 # mkfs.ext2 /dev/sda2
 # mkfs.ext4 /dev/vg01/rootfs
 ```
 ( **NOTE**: First for `Boot` and the second for `Root` )  

 11. Activate the `SWAP` logical partition by:  
 ```
 # mkswap /dev/vg01/swap
 # swapon /dev/vg01/swap
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
 11. Download `Stage 3` tarball from [HERE][gentoodownload] by `curl` command:  
 ```
 # curl -LO https://(your tarball link here)
 ```
 12. Unpack the `Stage 3` tarball by:  
 ```
 # tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
 ```
 ( **NOTE**: You can remove downloaded archive by: `rm /stage3-*.tar.*` )  

 13. Configuring compile options by:  
 ```
 # vim /mnt/gentoo/etc/portage/make.conf
 ```
 Add the `-march=native` and `-j4` there ( good choice is the number of CPUs / CPU cores in the system for -j ):  
 ```
 COMMON_FLAGS="-march=native -O2 -pipe"
 MAKEOPTS="-j4"
 ```
 ( **NOTE**: Read [THIS][makeopts] article also. )  

 14. Cause `mirrorselect` is absent on `System Rescue CD` we will do it manually. Official information about mirrors are [HERE][gentoomirrors]. Add into `/mnt/gentoo/etc/portage/make.conf` below the mirrors specific for your country. For `RU` it will be:  
 ```
 GENTOO_MIRRORS="https://mirror.yandex.ru/gentoo-distfiles/ https://gentoo-mirror.alexxy.name/"
 ```
 15. Create the `ebuild` repository:  
 ```
 # mkdir --parents /mnt/gentoo/etc/portage/repos.conf
 # cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
 # cat /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
 ```
 16. Copy `DNS` info by:  
 ```
 # cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
 ```
 ( **NOTE**: If you abort the installation you can start over from this step. )  

 17. Mounting the necessary file systems by.  
 ```
 # mount --types proc /proc /mnt/gentoo/proc
 # mount --rbind /sys /mnt/gentoo/sys
 # mount --make-rslave /mnt/gentoo/sys
 # mount --rbind /dev /mnt/gentoo/dev
 # mount --make-rslave /mnt/gentoo/dev
 ```
 18. Entering the new environment by:  
 ```
 # chroot /mnt/gentoo /bin/bash
 # source /etc/profile
 # export PS1="(chroot) ${PS1}"
 ```
 ( **NOTE**: Also you can make the [BASH][bash] script in root folder of future [Gentoo][gentoo] if you do this frequent. )  
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
 19. Don't forget to mount the `boot` partition by:  
 ```
 # mount /dev/sda2 /boot
 ```
 20. Install the [Gentoo][gentoo] `ebuild` repository snapshot from the `WEB` by:  
 ```
 # emerge-webrsync
 ```
 21. Purge the news if you are not going to read them by:  
 ```
 # eselect news read all
 # eselect news purge
 ```
 22. Choosing the right profile. `eselect profile set (correct number)` We will use `default` one. Check it by:  
 ```
 # eselect profile list
 ```
 23. Update the @world set:  
 ```
 # emerge --ask --verbose --update --deep --newuse @world
 ```
 ( **NOTE**: It may TAKE A LOT OF TIME depends on your hardware. You can skip it now. )  

 24. Set the `Timezone`. For `RU/Moscow` it will be:  
 ```
 # echo "Europe/Moscow" > /etc/timezone
 # emerge --config sys-libs/timezone-data
 ```
 25. Setup `VIM` cause I hate to use `Nano` by:  
 ```
 # emerge -a vim
 ```
 26. Now generate the locale. Add this into `/etc/locale.gen` for US ones:  
 ```
 en_US ISO-8859-1
 en_US.UTF-8 UTF-8
 ```
 27. Run this command:  
 ```
 # locale-gen
 ```
 28. Make `UTF-8` locale selection by:  
 ```
 # eselect locale list
 # eselect locale set (correct number)
 ```
 29. Reload `envirionment`:  
 ```
 # env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
 ```
 30. Install `LVM2` by:  
 ```
 # emerge -a sys-fs/lvm2
 ```
 31. Install the `Kernel` sources by:  
 ```
 # emerge -a sys-kernel/gentoo-sources
 # ls -l /usr/src/linux
 ```
 32. Create the configuration file for future `Kernel` compilation:  
 ```
 # cd /usr/src/linux
 # make defconfig
 # make menuconfig
 ```
 ( **NOTE**: For correct Kernel `LVM` configuration follow the [LVM official instruction][gentoolvm]. )  

 33. You can configure Kernel manually which is most effective way. Anyway, setup `genkernel` for future use. See below.  
 ```
 # emerge -a sys-kernel/genkernel
 # vim /etc/fstab
 ```
 Add this into `fstab` file. This based on default `LVM` configuration (pay attention to tabs between words):  
 ```
 /dev/sda2         /boot       ext2  defaults,noatime  0 2
 /dev/vg01/swap    none        swap  sw                0 0
 /dev/vg01/rootfs  /           ext4  noatime           0 1
 /dev/cdrom        /mnt/cdrom  auto  noauto,user       0 0
 ```
 34. Compile the `Kernel` by:  
 ```
 # make -j4 && make modules_install && make install
 ```
 ( **NOTE**: `-j4` for four CPU cores processor. )  

 35. Create `initramfs` by:  
 ```
 # genkernel --lvm --install initramfs
 ```
 ( **NOTE**: Check if `initramfs` was created by: `ls -la /boot` )  

 36. Add `hostname` into `/etc/conf.d/hostname`:  
 ```
 # Set the hostname variable to the selected host name
 hostname="gentoobox"
 ```
 37. Install for network configuration:  
 ```
 # emerge -a --noreplace net-misc/netifrc
 ```
 38. For `DHCP` add your `eth0` interface name into `/etc/conf.d/net`:  
 ```
 config_eth0="dhcp"
 ```
 39. Automatically start networking at boot:  
 ```
 # cd /etc/init.d
 # ln -s net.lo net.eth0
 # rc-update add net.eth0 default
 ```
 40. Add into `/etc/hosts`:  
 ```
 # This defines the current system and must be set
 127.0.0.1 gentoobox.localnetwork gentoobox localhost
 ```
 41. Set `ROOT` password:  
 ```
 # passwd
 ```
 42. Check out this files:  
 ```
 # vim /etc/rc.conf
 # vim /etc/conf.d/keymaps
 # vim /etc/conf.d/hwclock
 ```
 43. System logger setup:  
 ```
 # emerge -a app-admin/sysklogd
 # rc-update add sysklogd default
 # emerge -a app-admin/logrotate
 ```
 44. `Cron` daemon setup:  
 ```
 # emerge -a sys-process/cronie
 # rc-update add cronie default
 ```
 45. File indexing setup:  
 ```
 # emerge -a sys-apps/mlocate
 ```
 46. Remote access setup:  
 ```
 # rc-update add sshd default
 ```
 47. Installing a `DHCP` client:  
 ```
 # emerge -a net-misc/dhcpcd
 ```
 48. `Grub 2` should be compiled with device-mapper support. Do it by:  
 ```
 # echo 'sys-boot/grub:2 device-mapper' >> /etc/portage/package.use/package.use
 ```
 49. Setup `Grub 2` boot loader ( for `sda` in our case ):  
 ```
 # emerge --ask --verbose sys-boot/grub:2
 # grub-install /dev/sda
 ```
 50. Add `dolvm` for `GRUB_CMDLINE_LINUX_DEFAULT` variable into the `Grub 2` configuration file `/etc/default/grub` by:  
 ```
 # vim /etc/default/grub
 ```
 ( **NOTE**: Add `GRUB_CMDLINE_LINUX_DEFAULT="dolvm"` )  

 51. Then generate the `Grub 2` configuration file by:  
 ```
 # grub-mkconfig -o /boot/grub/grub.cfg
 ```
 ( **NOTE**: Ignore the `WARNINGS: /run/lvm/lvmetad.socket: connect failed` )  

 52. Setup almost complete. Reboot now:  
 ```
 # exit
 # cd
 # umount -l /mnt/gentoo/dev{/shm,/pts,}
 # umount -R /mnt/gentoo
 # reboot
 ```
 53. If first boot successful setup user by:  
 ```
 # useradd -m -G users,wheel,audio -s /bin/bash username
 # passwd username
 ```
 ( **NOTE**: Don't forget to unblock the wheel group, check if user can access root account and block the `root` account by: `sudo passwd -l root`. DO NOT login as `root` )  

### License  

This text has been written by ©2020 DimiG

[gentoo]:https://www.gentoo.org/
[gentoobook]:https://wiki.gentoo.org/wiki/Handbook:AMD64
[gentoomini]:https://www.gentoo.org/downloads/
[gentoodownload]:https://gentoo.org/downloads/
[gentoomirrors]:https://www.gentoo.org/downloads/mirrors/
[gentoolvm]:https://wiki.gentoo.org/wiki/LVM
[pc]:https://en.wikipedia.org/wiki/Personal_computer
[ethernet]:https://en.wikipedia.org/wiki/Ethernet
[guid]:https://en.wikipedia.org/wiki/GUID_Partition_Table
[parted]:https://en.wikipedia.org/wiki/GNU_Parted
[sysrescuecd]:https://www.system-rescue-cd.org/
[systemd]:https://en.wikipedia.org/wiki/Systemd
[swapsize]:https://itsfoss.com/swap-size/
[virtualmachine]:https://en.wikipedia.org/wiki/Virtual_machine
[lvm]:https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)
[bash]:https://en.wikipedia.org/wiki/Bash_(Unix_shell)
[makeopts]:https://blogs.gentoo.org/ago/2013/01/14/makeopts-jcore-1-is-not-the-best-optimization/
