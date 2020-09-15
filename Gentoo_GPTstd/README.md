Gentoo_GPTstd
=============
This is `HowTo` step by step instruction of [Gentoo][gentoo] installation on [PC hardware][pc]. There is a nice official documentation [HERE][gentoobook], but the aim of this `HowTo` is short step by step command for faster installation. The brief explanation is look onto official [Gentoo WEB site][gentoo].  

The way of installation specific for me and can be tweaked further.  

### Installation procedures  

First of all check out your [PC][pc] has [Ethernet][ethernet] connection to the Internet. The [Gentoo][gentoo] is downloadable by Internet and without it is not possible to install.  

To make partitions on [GPT][guid] partition disk I prefer to use command line [parted][parted] program as described in [official documentation][gentoobook].  

To install [Gentoo][gentoo] you can use official [minimal installation CD][gentoomini] which is recommended, but this CD is NOT possible to boot by [PXE][pxe]. This is not convenient for me. So, in this `HowTo` I will use `third party` [System Rescue Cd][sysrescuecd] which is more friendly for installation.  

#### Pre-installation:  

 1.  Boot the [System Rescue CD][sysrescuecd].  
 2.  Check if you have Internet connection by `ip a` and `ping google.com` command. If NOT you have to solve this problem before installation.  
 3.  If you wish to connect to your installation by `SSH` from other computer you must change the password for `root` by `passwd` command and stop the `iptables` firewall which is running on this CD by `# systemctl stop iptables`. Then connect to your PC by IP from `ip a` command.  
 4.  If you successfully connect to your system by `SSH` as `root` - you are on the way to the next steps.  

#### General Installation procedure (standard install on [GPT][guid]):  

 1.  Check you hard drive configuration by `# lsblk -f` command.  
 Be careful and DON'T format the partition with important data. **YOU WERE WARNED**!  
 Based on official documentation the standard partitioning scheme is:  
 ```
 /dev/sda1 (bootloader) 2M   BIOS boot partition
 /dev/sda2 ext2         128M Boot/EFI system partition
 /dev/sda3 (swap)       512M Swap partition
 /dev/sda4 ext4         MAX  Root partition
 ```
 2.  Run the: `# parted -a optimal /dev/sda` for `sda` hard drive (name depends on your system).  
 3.  Clean up the disk and make `GPT` label by: `(parted) mklabel gpt`.  
 4.  Setup the UNIT by: `(parted) unit mib`  
 5.  Add additional commands and check result by `print` command. Then `quit` from there:  
 ```
 (parted) mkpart primary 1 3
 (parted) name 1 grub
 (parted) set 1 bios_grub on
 (parted) mkpart primary 3 131
 (parted) name 2 boot
 (parted) mkpart primary 131 643
 (parted) name 3 swap
 (parted) mkpart primary 643 -1
 (parted) name 4 rootfs
 (parted) set 2 boot on
 ```
 ( **NOTE**: The `SWAP` partition size depends on you `RAM`. Consult [HERE][swapsize]. The `SWAP` partition is possible to enable after installation. Come back to this later )  

 6.  Format created partitions by:  
 ```
 # mkfs.ext2 /dev/sda2
 # mkfs.ext4 /dev/sda4
 ```
 7.  Create the mounting point by:  
 ```
 # mkdir /mnt/gentoo
 ```
 8.  Mount the `ROOT` partition:  
 ```
 # mount /dev/sda4 /mnt/gentoo
 ```
 9.  Check out the current date and set up it by:  
 ```
 # ntpd -q -g
 ```
 ( **NOTE**: Sure, you can do it manually by `date` command )  

 10. Go to the [Gentoo][gentoo] folder:  
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
 13. Configuring compile options by:  
 ```
 # vim /mnt/gentoo/etc/portage/make.conf
 ```
 Add the `-march=native` and `-j9` there ( good choice is the number of CPUs / CPU cores in the system plus one for -j ):  
 ```
 COMMON_FLAGS="-march=native -O2 -pipe"
 MAKEOPTS="-j9"
 ```
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
 16. If you abort the installation you can start over from this step. Copy `DNS` info by:  
 ```
 # cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
 ```
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
 # eselect news purge
 ```
 22. Choosing the right profile. `# eselect profile set (correct number)` We will use `default` one. Check it by:  
 ```
 # eselect profile list
 ```
 23. Updating the @world set. It may takes a lot of time depends on your hardware:  
 ```
 # emerge --ask --verbose --update --deep --newuse @world
 ```
 24. Set the `Timezone`. For `RU/Moscow` it will be:  
 ```
 # echo "Europe/Moscow" > /etc/timezone
 # emerge --config sys-libs/timezone-data
 ```
 25. Setup `VIM` cause I don't use `Nano` by:  
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
 30. Install the `Kernel` sources by:  
 ```
 # emerge -a sys-kernel/gentoo-sources
 # ls -l /usr/src/linux
 ```
 31. Install `lspci` command and create the configuration file for future `Kernel` compilation:  
 ```
 # emerge -a sys-apps/pciutils
 # cd /usr/src/linux
 # make menuconfig
 ```
 32. You can configure Kernel manually which is most effective way, but here I will do it automatically. See below.  
 ( **NOTE**: For manual compilation do: `# make -j9 && make -j9 modules_install && make -j9 install` )  
 ```
 # emerge -a sys-kernel/genkernel
 # vim /etc/fstab
 ```
 Add this into `fstab` file (pay attention to tabs between words):  
 ```
 /dev/sda2 /boot ext2 defaults 0 2
 ```
 33. Compile the `Kernel`. For `LVM` don't forget `--lvm` as an argument:  
 ```
 # genkernel all
 ```
 34. Check if it was created by:  
 ```
 # ls /boot/vmlinu* /boot/initramfs*
 ```
 35. Install firmware by:  
 ```
 # emerge -a sys-kernel/linux-firmware
 ```
 36. Setup the `fstab` file. Here is example ( pay attention to tabs between words ):  
 ```
 /dev/sda2  /boot      ext2 defaults,noatime 0 2
 /dev/sda3  none       swap sw               0 0
 /dev/sda4  /          ext4 noatime          0 1
 /dev/cdrom /mnt/cdrom auto noauto,user      0 0
 ```
 37. Activate the `swap` partition:  
 ```
 # mkswap /dev/sda3
 # swapon /dev/sda3
 ```
 38. Add `hostname` into `/etc/conf.d/hostname`:  
 ```
 # Set the hostname variable to the selected host name
 hostname="gentoobox"
 ```
 39. Install for network configuration:  
 ```
 # emerge -a --noreplace net-misc/netifrc
 ```
 40. For `DHCP` add your interface name into `/etc/conf.d/net`:  
 ```
 config_eth0="dhcp"
 ```
 41. Automatically start networking at boot:  
 ```
 # cd /etc/init.d
 # ln -s net.lo net.eth0
 # rc-update add net.eth0 default
 ```
 42. Add into `/etc/hosts`:  
 ```
 # This defines the current system and must be set
 127.0.0.1     gentoobox.localnetwork gentoobox localhost
 ```
 43. Set `ROOT` password:  
 ```
 # passwd
 ```
 44. Check out this files:  
 ```
 # vim /etc/rc.conf
 # vim /etc/conf.d/keymaps
 # vim /etc/conf.d/hwclock
 ```
 45. System logger setup:  
 ```
 # emerge -a app-admin/sysklogd
 # rc-update add sysklogd default
 ```
 46. `Cron` daemon setup:  
 ```
 # emerge -a sys-process/cronie
 # rc-update add cronie default
 ```
 47. File indexing setup:  
 ```
 # emerge -a sys-apps/mlocate
 ```
 48. Remote access setup:  
 ```
 # rc-update add sshd default
 ```
 49. Installing a `DHCP` client:  
 ```
 # emerge -a net-misc/dhcpcd
 ```
 50. Boot loader setup ( `Grub 2` for `sda` in our case ):  
 ```
 # emerge -a --verbose sys-boot/grub:2
 # grub-install /dev/sda
 # grub-mkconfig -o /boot/grub/grub.cfg
 ```
 51. Setup almost complete. Reboot and pray:  
 ```
 # exit
 # cd
 # umount -l /mnt/gentoo/dev{/shm,/pts,}
 # umount -R /mnt/gentoo
 # reboot
 ```
 52. If first boot successful setup user and remove unnecessary files by:  
 ```
 # useradd -m -G users,wheel,audio -s /bin/bash username
 # passwd username
 # rm /stage3-*.tar.*
 ```
 ( **NOTE**: Don't forget to unblock the wheel group, check if user can access root account and block the `root` account by: `# sudo passwd -l root`. DO NOT login as `root` )  

### License  

This text has been written by ©2020 DimiG

[gentoo]:https://www.gentoo.org/
[gentoobook]:https://wiki.gentoo.org/wiki/Handbook:AMD64
[gentoomini]:https://www.gentoo.org/downloads/
[gentoodownload]:https://gentoo.org/downloads/
[gentoomirrors]:https://www.gentoo.org/downloads/mirrors/
[pc]:https://en.wikipedia.org/wiki/Personal_computer
[ethernet]:https://en.wikipedia.org/wiki/Ethernet
[guid]:https://en.wikipedia.org/wiki/GUID_Partition_Table
[parted]:https://en.wikipedia.org/wiki/GNU_Parted
[pxe]:https://en.wikipedia.org/wiki/Preboot_Execution_Environment
[sysrescuecd]:https://www.system-rescue-cd.org/
[swapsize]:https://itsfoss.com/swap-size/
