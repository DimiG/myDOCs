ArchMBP_UEFIstd
===============
This is `HowTo` step by step instruction of [Arch Linux][archlnx] installation on [Mac Book Pro][mbp] It is NOT for **noobs** and you MUST know the Mac OS system administration, know well console actions in Mac OS, [BASH][bash] and [Linux][lnx] in general, and most important understand what you do. ;) (**May contain incorrect commands and NOT complete yet...**).  

The [Apple][apple] devices use modern [UEFI][uefi] (Unified Extensible Firmware Interface) the [BIOS][bios] (Basic Input/Output System) replacement. So, need to use [GPT][guid] (GUID Partition Table) partition table on Hard Drive to install Arch Linux along with installed [Mac OS][macos]. Here is how.  

### Installation procedures  

First of all check out your [Mac Book Pro][mbp] (MBP) has [Ethernet][ethernet] connection to the Internet. The [Arch Linux][archlnx] distribution download system by Internet. `WiFi` is NOT easy to setup, so use [Ethernet][ethernet] port if exists or use `USB to Ethernet` adapter or something.  

To make partitions on [GPT][guid] partition disk I prefer to use Curses-based `GUID` partition table (GPT) manipulator [cgdisk][cgdisk] or command line [gdisk][gdisk].  

The [Arch Linux installation][archinst] described on [official WEB site][archinst] well. Here is my way to do it.  

#### Pre-installation:  

 1.  Prepare the free space on your hard disk (HDD/SSD) for Linux by native Mac OS disk utility. I highly don't recommend move Mac OS partitions by third party utils. Always Do backup of Mac OS system before any partition manipulations. You may lose everything! YOU WERE WARNED! How to do it you can easily find in the Internet.  
 2.  Prepare the boot-able `Arch Linux` thumb drive by downloaded ISO from [HERE][archdl].  
 3.  Reboot your `MBP` with **Alt** key pressed and chose `EFI Boot` and boot into your `Arch Linux` installer.  
 4.  Check if you have booted in EFI mode by: `# efivar -l` command. If you see variables it is OK. You are in EFI mode.  
 5.  Check if you have Internet connection by `ip a` and `ping google.com` command. If NOT you have to solve this problem before installation.  
 6.  If you wish to connect to `MBP` from other computer by `SSH` you can run `SSH` daemon by: `# systemctl start sshd` and connect to it.  
 7.  Update package indexes by: `# pacman -Syyy`  

#### General Installation procedure (standard install on [UEFI][uefi]):  

 1.  Check you hard drive configuration by `# lsblk` command. Usually you have three partitions: `sda1` is EFI, `sda2` is `macOS` system, `sda3` is Mac OS recovery. Cause the EFI exists we will use it for `Arch Linux` installation. What you need in this case is `ROOT` Linux partition and `HOME` partition. I don't create the `SWAP` partition and prefer the more modern method to use for `SWAP` the special created file.  
 2.  Start up the `gdisk` by `# gdisk /dev/sda` and create the Linux `ROOT` partition by `n` and `+30G` command. Also create `HOME` partition by `n` for FREE EMPTY SPACE. Check the table by `p` command if it is OK - write it by `w` command. Now if you give `# lsblk` command you MUST see the `hda4` and `sda5` partition for `Arch Linux` installation. Your EFI should be near `300 Mb` size and your Linux `ROOT` partition `30 Gb` or larger.  
 3.  DON'T TOUCH the EFI partition and DO NOT format it. It contains special files for Mac OS boot. Only format the `sda4` and `sda5` by: `# mkfs.ext4 /dev/sda4` `# mkfs.ext4 /dev/sda5`. PAY ATTENTION if you have other partition allocation this `HowTo` is NOT FOR YOU!  
 4.  Next update the system clock by:  
 ```
 # timedatectl set-ntp true
 ```
 5.  Mount your created Linux partition by:  
 ```
 # mount /dev/sda4 /mnt
 ```
 6.  Create the `home` and mount it:  
 ```
 # mkdir /mnt/home && mount /dev/sda5 /mnt/home
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
 # pacman -Syyy
 ```
 12. I won't use `GRUB` here cause had NO SUCCESS to start it. Also I will use `LTS` kernel as recommended by others. We will install additional packages by:  
 ```
 # pacman -S efibootmgr dosfstools openssh dhcpcd os-prober \
   mtools linux-headers linux-lts linux-lts-headers vim vi
 ```
 13. Next setup the your `Time Zone` (Must be your REGION and CITY) by:  
 ```
 # ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
 ```
 14. Run the `hwclock` to generate `/etc/adjtime` by:  
 ```
 # hwclock --systohc --utc
 ```
 15. Make localization by uncommenting `en_US.UTF-8` or other needed locales in `/etc/locale.gen` and generate them with `# locale-gen` command.  
 16. Add `LANG=en_US.UTF-8` into `/etc/locale.conf`  
 17. If you set the keyboard layout, make the changes persistent in `vconsole.conf` by adding: `KEYMAP=ru4` or anything else.  
 18. Create the host name (chose your name) by:  
 ```
 # echo 'myhostname' > /etc/hostname
 ```
 19. Add matching names in hosts:  
 ```
 127.0.0.1	localhost
 ::1		localhost
 127.0.1.1	myhostname.localdomain	myhostname
 ```
 ( **Note**: If the system has a permanent IP address, it should be used instead of 127.0.1.1 )  

 20. Set the root password:  
 ```
 # passwd
 ```
 21. Create the user:  
 ```
 # useradd -m -g users -G wheel -s /bin/bash good_user
 ```
 22. Change user password:  
 ```
 # passwd good_user
 ```
 23. Enable SSH service if necessary by:  
 ```
 # systemctl enable sshd.service
 ```
 24. Enable DHCP client service by:  
 ```
 # systemctl enable dhcpcd.service
 ```
 25. Create the SWAP file:  
 ```
 # fallocate -l 2G /swapfile
 # chmod 600 /swapfile
 # mkswap /swapfile
 # echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
 ```
 26. As boot loader for Mac OS platform I recommend to use `systemd-boot`:  
   - Copy all files from `/boot/` to `/root/tmp` (vmlinuz-linux-lts, initramfs-linux-lts.img)  
   - Mount EFI into `/boot` by:  
   ```
   # mount /dev/sda1 /boot
   ```
   - Get `ROOT` `PARTUUID` by:  
   ```
   # blkid -s PARTUUID -o value /dev/sda4
   ```
   - Install it by:  
   ```
   # bootctl install
   ```
   - Clean and add this text into `/boot/loader/loader.conf`:  
   ```
   default  arch
   timeout  4
   editor   0
   ```
   - Create the file and add this text into `/boot/loader/entries/arch.conf` (change number to yours received by `blkid` above):  
   ```
   title    Arch Linux
   linux    /vmlinuz-linux-lts
   initrd   /initramfs-linux-lts.img
   options  root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw
   ```
   - Copy files back from `/root/tmp` into `/boot`  
 27. Exit from Arch Linux CHROOT by:  
 ```
 # exit
 ```
 28. Unmount all by:  
 ```
 # umount -R /mnt
 ```
 29. Reboot and PRAY... :) If you will see the boot loader and `Arch Linux` will boot - you are LUCKY  

**NOTE**: For security reasons I suggest lock the `ROOT` account by: `$ sudo passwd -l root` and DO NOT login as `ROOT`  

### License  

This code has been written by ©2019 DimiG  

[archlnx]:https://www.archlinux.org
[archinst]:https://wiki.archlinux.org/index.php/installation_guide
[archdl]:https://www.archlinux.org/download/
[apple]:https://www.apple.com
[uefi]:https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface
[bios]:https://en.wikipedia.org/wiki/BIOS
[guid]:https://en.wikipedia.org/wiki/GUID_Partition_Table
[macos]:https://en.wikipedia.org/wiki/MacOS
[mbp]:https://en.wikipedia.org/wiki/MacBook_Pro
[ethernet]:https://en.wikipedia.org/wiki/Ethernet
[cgdisk]:https://linux.die.net/man/8/cgdisk
[gdisk]:https://linux.die.net/man/8/gdisk
[bash]:https://en.wikipedia.org/wiki/Bash_(Unix_shell)
[lnx]:https://en.wikipedia.org/wiki/Linux
[systemdboot]:https://wiki.archlinux.org/index.php/Systemd-boot
