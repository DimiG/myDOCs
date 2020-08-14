ArchEEEPC_MBRlvm
================
This is `HowTo` **step by step** instruction of [Arch Linux][archlnx] installation on [EeePC 900][eeepc] series. This uses method of [LVM][lvm] installation on [MBR][mbr] disk. I'm NOT the [Arch Linux][archlnx] expert, so my experiments may NOT 100% correct. Anyway, what I am writing here has been tested on real hardware.  

This small computer was very popular in **2007**. To use this instruction you **MUST** know how to operate with [Linux][lnx] in command line and know well shells and commands. The most important is to understand what you do. ;) (**May contain incorrect commands and NOT complete yet...**).  

The reason of these `HowTo`'s cause information about it is fragmented and scattered across the Internet. [Arch Linux][archlnx] installation is tricky. The [EeePC 900][eeepc] I used for installation is [Linux model][lnx] model, so problems with drivers is not so strong how with [Windows][windows] model. It contains two [SSD][ssd] like drives. One is `4Gb` size and the other one is `16Gb`. It has `WiFi`, one [Ethernet][ethernet] socket and two `USB 2.0` ports. You can boot from internal [SD][sd] flash card.  

Because of this unit has `i686` **32bit** processor on board and [Arch Linux][archlnx] officially support only the **64bit**, you need to download community maintained `distro` and setup it on [SD][sd] card. You can download it from [HERE][archdl].  

### Installation procedures

First of all check out your [EeePC 900][eeepc] (EeePC) has [Ethernet][ethernet] connection to the Internet. The [Arch Linux][archlnx] distribution download the system by Internet. `WiFi` is **NOT** easy to setup, so use [Ethernet][ethernet] port if exists or use `USB to Ethernet` adapter or something.  

For system installation I have decided to use small `4Gb` disk for `BOOT` and [SWAP][swap], the big `16Gb` one for [LVM][lvm] partition. The small disk is little bit slower compare to bigger one.  

Cause this old hardware has [BIOS][bios] on board, the internal disks **MUST BE** formatted as [MBR][mbr].  

The [Arch Linux installation][archinst] described on [official WEB site][archinst]. Here is my way to do it.  

#### Pre-installation:

 1.  Prepare the boot-able `Arch Linux` [SD card][sd] by downloaded `ISO` from [HERE][archdl]. I use [dd][dd] utility for that.  
 2.  Insert your SD card in the slot and boot from it. Push `Esc` to call [BIOS][bios] boot menu and chose SD card as boot-able device.  
 3.  Just after installer boot check if you have Internet connection by `ip a` and `ping google.com` command. If NOT you have to solve this problem before installation.  
 4.  If you wish to connect to [EeePC][eeepc] from other computer by `SSH` you can run `SSH` daemon by: `# systemctl start sshd`, give a `# passwd`, set `ROOT` password and connect with this password to device by [SSH][ssh].  
 5.  Update package indexes by: `# pacman -Syy`  

#### General Installation procedure (LVM install on MBR):

 1.  Check you hard drives configuration by `# lsblk` command. Usually you have two disks: `sda` and `sdb`. The first one is smaller one and we will use it for `BOOT` and [SWAP][swap]. The second one is large disk and we will use it for [LVM][lvm] which will hold `ROOT` and `HOME`.  
 2.  Cause `EeePC 900` has `BIOS` and we will make `MBR` partitions I will use the classical [fdisk][fdisk] utility. Start up the `fdisk` by `# fdisk /dev/sda` and create the Linux `BOOT` partition by `n` and `+260M` command. Also create `SWAP` partition by `n` for FREE EMPTY SPACE. Set the boot-able flag for `sda1` by `a` command and swap type `82` by `t` for `sda2`. Check the table by `p` command if it is OK - write it by `w` command.  
 3.  Run [fdisk][fdisk] again with: `# fdisk /dev/sdb` and create Linux `LVM` partition on it by `n` command. Set type by `t` to `8e`. Check the table by `p` command if it is OK - write it by `w` command. Now if you give `# lsblk` command you MUST see the `hda` and `sdb` partitions for `Arch Linux` installation.  
 4.  Format the `sda1` by: `# mkfs.ext2 /dev/sda1`.  
 5.  Next update the system clock by:  
 ```
 # timedatectl set-ntp true
 ```
 6.  Create the [LVM][lvm] for [SSD][ssd] disk by:  
 ```
 # pvcreate --dataalignment 1m /dev/sdb1
 # vgcreate vg0 /dev/sdb1
 # lvcreate -L 10GB vg0 -n lv_root
 # lvcreate -l 100%FREE vg0 -n lv_home
 # modprobe dm_mod
 # vgscan
 # vgchange -ay
 # mkfs.ext4 /dev/vg0/lv_root
 # mkfs.ext4 /dev/vg0/lv_home
 # mount /dev/vg0/lv_root /mnt
 # mkdir /mnt/home
 # mkdir /mnt/boot
 # mount /dev/sda1 /mnt/boot
 # mount /dev/vg0/lv_home /mnt/home
 ```
 7.  Install the `Arch Linux` by:  
 ```
 # pacstrap -i /mnt base base-devel
 ```
 8.  Let's create the `fstab` file by:  
 ```
 # genfstab -Up /mnt >> /mnt/etc/fstab
 ```
 9.  Check if it was really created by:  
 ```
 # cat /mnt/etc/fstab
 ```
 10. Chroot into the installed Arch by:  
 ```
 # arch-chroot /mnt
 ```
 11. Update package indexes by:  
 ```
 # pacman -Syy
 ```
 12. We will use `LTS` kernel as recommended by others. We will install additional packages by:  
 ```
 # pacman -S openssh grub-bios linux-headers linux-lts linux-lts-headers dosfstools \
   dhcpcd os-prober mtools lvm2 vim vi
 ```
 13. Edit `/etc/mkinitcpio.conf` and add `lvm2` in between `block` and `filesystems`.  
 14. Create boot files by:  
 ```
 # mkinitcpio -p linux-lts
 ```
 15. Next setup the your `Time Zone` (Must be your REGION and CITY) by:  
 ```
 # ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
 ```
 16. Run the `hwclock` to generate `/etc/adjtime` by:  
 ```
 # hwclock --systohc --utc
 ```
 17. Edit the `locale.gen` file and uncomment `en_US.UTF-8` or other needed locales in `/etc/locale.gen` by:  
 ```
 vim /etc/locale.gen
 ```
 18. Generate locale by:  
 ```
 # locale-gen
 ```
 19. Set locale by:  
 ```
 # localectl set-locale LANG="en_US.UTF-8"
 ```
 20. Create the host name (chose your name) by:  
 ```
 # echo 'myhostname' > /etc/hostname
 ```
 21. Add matching names in hosts:  
 ```
 127.0.0.1	localhost
 ::1		localhost
 127.0.1.1	myhostname.localdomain	myhostname
 ```
 ( **Note**: If the system has a permanent IP address, it should be used instead of 127.0.1.1 )  

 22. Set the root password:  
 ```
 # passwd
 ```
 23. Enable root logon via ssh.  
 24. Create the user:  
 ```
 # useradd -m -g users -G wheel -s /bin/bash good_user
 ```
 25. Change user password:  
 ```
 # passwd good_user
 ```
 26. Enable SSH service if necessary by:  
 ```
 # systemctl enable sshd.service
 ```
 27. Enable DHCP client service by:  
 ```
 # systemctl enable dhcpcd.service
 ```
 28. Install [GRUB][grub] by:  
 ```
 # grub-install --target=i386-pc --recheck /dev/sda
 # mkdir /boot/grub/locale
 # cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
 # grub-mkconfig -o /boot/grub/grub.cfg
 ```
 29. Create the [SWAP][swap] partition:  
 ```
 # mkswap /dev/sda2
 # swapon /dev/sda2
 ```
 30. To enable this [SWAP][swap] partition on boot, add an entry to `/etc/fstab`. Remember the UUID and replace it to actual one:  
 ```
 # echo 'UUID=device_UUID none swap defaults 0 0' | tee -a /etc/fstab
 ```
 31. If you have [SSD][ssd] disk (here we have), change `/etc/fstab` by adding `discard` like this:  
 ```
 UUID=<UUID> / ext4 defaults,noatime,discard 0 2
 ```
 32. Add to `/etc/fstab`:  
 ```
 # echo 'tmpfs /tmp tmpfs nodev,nosuid,size=1G 0 0' | tee -a /etc/fstab
 ```
 33. Exit from [Arch Linux][archlnx] CHROOT by:  
 ```
 # exit
 ```
 34. Unmount all by:  
 ```
 # umount -R /mnt
 ```
 35. Reboot and hope you will see the [GRUB][grub] boot screen... :)  
 
 36. Setup additional software by [pacman][pacman] when boot.  

**NOTE**: For security reasons I suggest lock down the `ROOT` account by: `$ sudo passwd -l root` and DO NOT login as `ROOT`.  

### License

This text has been written by ©2019 DimiG

[archlnx]:https://www.archlinux.org
[eeepc]:https://en.wikipedia.org/wiki/Asus_Eee_PC
[lnx]:https://en.wikipedia.org/wiki/Linux
[bash]:https://en.wikipedia.org/wiki/Bash_(Unix_shell)
[windows]:https://en.wikipedia.org/wiki/Microsoft_Windows
[sd]:https://ru.wikipedia.org/wiki/Secure_Digital
[ssd]:https://en.wikipedia.org/wiki/Solid-state_drive
[archdl]:https://www.archlinux32.org/download/
[ethernet]:https://en.wikipedia.org/wiki/Ethernet
[archinst]:https://wiki.archlinux.org/index.php/Installation_guide
[lvm]:https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)
[mbr]:https://en.wikipedia.org/wiki/Master_boot_record
[bios]:https://en.wikipedia.org/wiki/BIOS
[dd]:https://wiki.archlinux.org/index.php/Dd
[ssh]:https://en.wikipedia.org/wiki/Secure_Shell
[fdisk]:https://en.wikipedia.org/wiki/Fdisk
[grub]:https://wiki.archlinux.org/index.php/GRUB
[swap]:https://wiki.archlinux.org/index.php/Swap
[uuid]:https://en.wikipedia.org/wiki/Universally_unique_identifier
[pacman]:https://www.archlinux.org/pacman/
