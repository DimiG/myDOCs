ArchMBP_UEFIlvm
===============
This `HowTo` step by step instruction of [Arch Linux][archlnx] installation on [Mac Book Pro][mbp] This document is **NOT** for **noobs** and you **MUST** to know the [Mac OS][macos] system administration, to know well the console actions in [Mac OS][macos], [BASH][bash] and [Linux][lnx] in general, and most important UNDERSTAND what you do. ;) (**May contain incorrect commands and NOT yet complete...**).  

The [Apple][apple] devices use modern [UEFI][uefi] (Unified Extensible Firmware Interface) the [BIOS][bios] (Basic Input/Output System) replacement. So, need to use [GPT][guid] (GUID Partition Table) partition table on Hard Drive to install Arch Linux along with installed [Mac OS][macos].  

Despite using [UEFI][uefi], the MacBook's native EFI bootloader does not use the EFI partition for booting. Instead, it looks for `.efi` files inside all the partitions in internal and external drives and shows them as possible boot options if certain conditions are satisfied.  

This `HowTo` describes the method of installation without any third party boot manager and with modern [LVM][lvm] support. Tested on real [MBP mid 2009 (5,5)][mbp] model. Here is how.  

### Installation procedures  

First of all check out your [Mac Book Pro][mbp] (MBP) has [Ethernet][ethernet] connection to the *Internet*. The [Arch Linux][archlnx] distribution downloads system by Internet. `WiFi` is NOT easy to setup, so use [Ethernet][ethernet] hardware port if exists or use `USB to Ethernet` adapter or something **;)**  

To make partitions on [GPT][guid] partition disk I prefer to use curses-based `GUID` partition table (GPT) manipulator [cgdisk][cgdisk] or command line [gdisk][gdisk].  

The [Arch Linux installation][archinst] described on [official WEB site][archinst] well. *Additional official documentation for Macs* located [HERE][archmacinst]. My way to do it shown below. I prefer to use [vim][vim] as text editor.  

#### Pre-installation:  

 1.  Prepare the FREE SPACE on your hard drive (HDD/SSD) for Linux by native Mac OS disk utility. I highly DON'T recommend move the Mac OS partitions by third party utils. ALWAYS DO BACKUP of your Mac OS important data before any partition manipulations. You MAY LOSE everything! YOU WERE WARNED! How to do it you can easily find in the Internet.  
 2.  Prepare the boot-able `Arch Linux` thumb drive by downloaded ISO from [HERE][archdl].  
 3.  Reboot your `MBP` with **Alt** key pressed and chose `EFI Boot` and boot into your `Arch Linux` installer.  
 4.  Check if you have booted in EFI mode by: `# efivar -l` command. If you see variables it is OK. You are in EFI mode.  
 5.  Check if you have Internet connection by `ip a` and `ping 8.8.8.8` command. If NOT you have to solve this problem before installation.  
 6.  Update the system clock by: `# timedatectl set-ntp true`  
 7.  If you wish to connect to `MBP` from other computer by `SSH` you can run `SSH` daemon by: `# systemctl start sshd`, change password for `root` and connect to it.  
 8.  Correct the [pacman][pacman] mirror list depends on your country inside this file: `vim /etc/pacman.d/mirror`.  
 9.  Update package indexes by: `# pacman -Syy`  

#### General Installation procedure (LVM install on [UEFI][uefi]):  

 1.  Check you hard drive configuration by `# lsblk` command. Usually you have three partitions: `sda1` is EFI, `sda2` is `macOS` system, `sda3` is `Mac OS recovery`. We will add a new [HFS+][hfsplus] formatted partition for `EFI BOOT` and use it for `Arch Linux` installation. The `ROOT` and `HOME` partition will be located in separate [LVM][lvm] partition. I DON'T create the `SWAP` partition and prefer to use the file for `SWAP`.  
 2.  Start up the `gdisk` by `# gdisk /dev/sda` and create the Mac OS `BOOT` partition by `n` and `+250M` command. OS X likes to see a **128Mb** gap after partitions, so when you create the first partition after the last OS X partition, type in **+128M** when `gdisk` asks for the first sector of new partition. Not necessary, but recommended. Type for [HFS+][hfsplus] is: `af00`. Also create `LVM` partition by `n` for FREE EMPTY SPACE. Type for [LVM][lvm] is: `8e00`. Check the table by `p` command if it is OK - write it by `w` command. Now if you give `# lsblk -f` command you MUST see the `sda4` and `sda5` partitions for `Arch Linux` installation. Your `EFI BOOT` should be near `250 Mb` size and your Linux [LVM][lvm] partition `30 Gb` or larger.  
 3.  Since the `Arch` installation **ISO** does not contain the `hfsprogs` package, but it is located in [AUR][aur], we need to compile and install it in the `chrooted` installation environment before proceeding with formatting the new partition as [HFS+][hfsplus]. So, lets create the [LVM][lvm] and format it:  
 ```
 # pvcreate /dev/sda5
 ( if SSD disk: # pvcreate --dataalignment 1m /dev/sda5 )
 # vgcreate vg0 /dev/sda5
 # lvcreate -L 25GB vg0 -n lv_root
 # lvcreate -l 100%FREE vg0 -n lv_home
 # modprobe dm_mod
 # vgscan
 # vgchange -ay
 # mkfs.ext4 /dev/vg0/lv_root
 # mkfs.ext4 /dev/vg0/lv_home
 # mount /dev/vg0/lv_root /mnt
 # mkdir /mnt/home
 # mkdir /mnt/boot
 # mount /dev/vg0/lv_home /mnt/home
 ```
 4.  Install the `Arch Linux` by:  
 ```
 # pacstrap -i /mnt base
 ```
 5.  Chroot into the installed `Arch` by:  
 ```
 # arch-chroot /mnt
 ```
 6.  Update package indexes by:  
 ```
 # pacman -Syy
 ```
 7.  Let's install necessary packages for `hfsprogs` compilation from `AUR`:  
 ```
 # pacman -S base-devel git vim vi
 ```
 8.  By security reasons you can't compile programs as `root`, so let's create the user and make compilation from there:  
 ```
 # useradd -m -g users -G wheel -s /bin/bash good_user
 # passwd good_user
 # EDITOR=vim visudo
 ( uncomment the wheel group )
 # sudo -iu good_user
 ```
 9.  Then create `tmp` directory in your home and go there:  
 ```
 $ mkdir tmp && cd tmp
 ```
 10. Git clone `Yay` `AUR` packet manager and compile it:  
 ```
 $ git clone https://aur.archlinux.org/yay-bin.git && cd yay-bin
 $ makepkg -si
 ```
 11. Then compile and install `hfsprogs` by `Yay`:  
 ```
 $ yay -S hfsprogs && exit
 ```
 12. If installation of `hfsprogs` passed good format the `/dev/sda4` by:  
 ```
 # mkfs.hfsplus /dev/sda4 -v "Arch Linux"
 ```
 13. Then mount it into `/boot`:  
 ```
 # exit
 # mount -t hfsplus -o force,rw /dev/sda4 /mnt/boot
 ```
 14. Let's create the `fstab` file by:  
 ```
 # genfstab -Up /mnt >> /mnt/etc/fstab
 ```
 15. Check if it was really created by:  
 ```
 # cat /mnt/etc/fstab
 ```
 16. Add option `force` into `/boot` option of `fstab` or otherwise it will be mounted as **read only** after reboot.  
 17. Chroot into the installed `Arch` again by:  
 ```
 # arch-chroot /mnt
 ```
 18. Let's install additional software into `chroot`:  
 ```
 # pacman -S openssh grub efibootmgr linux-headers linux-lts linux-lts-headers dosfstools \
   dhcpcd os-prober mtools lvm2
 ```
 19. Edit `/etc/mkinitcpio.conf` and add `lvm2` between `block` and `filesystems`.  
 20. Regenerate the `initramfs` while `chrooted`:  
 ```
 # mkinitcpio -p linux-lts
 ```
 21. Next setup your `Time Zone` (Must be your REGION and CITY) by:  
 ```
 # ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
 ```
 22. Run the `hwclock` to generate `/etc/adjtime` by:  
 ```
 # hwclock --systohc --utc
 ```
 23. Edit the `locale.gen` file and uncomment `en_US.UTF-8` or other needed locales in `/etc/locale.gen` by:  
 ```
 # vim /etc/locale.gen
 ```
 24. Generate locale by:  
 ```
 # locale-gen
 ```
 25. Set locale by:  
 ```
 # localectl set-locale LANG="en_US.UTF-8"
 ```
 26. Create the host name (chose your name) by:  
 ```
 # echo 'myhostname' > /etc/hostname
 ```
 27. Add matching names in hosts:  
 ```
 127.0.0.1	localhost
 ::1		localhost
 127.0.1.1	myhostname.localdomain	myhostname
 ```
 ( **Note**: If the system has a permanent IP address, it should be used instead of 127.0.1.1 )  
 28. Set the root password:  
 ```
 # passwd
 ```
 29. Enable SSH service if necessary by:  
 ```
 # systemctl enable sshd.service
 ```
 30. Enable DHCP client service by:  
 ```
 # systemctl enable dhcpcd.service
 ```
 31. Create the **SWAP** file:  
 ```
 # fallocate -l 2G /swapfile
 # chmod 600 /swapfile
 # mkswap /swapfile
 # echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
 ```
 32. Create a dummy `mach_kernel` by:  
 ```
 # touch /boot/mach_kernel
 # mkdir -p /boot/EFI/arch && touch /boot/EFI/arch/mach_kernel
 ```
 33. The following steps install the [GRUB][grub] UEFI application to `/boot/EFI/arch/System/Library/CoreServices/boot.efi` and install its modules to `/boot/grub/x86_64-efi`:  
 ```
 # grub-install --target=x86_64-efi --efi-directory=/boot
 ```
 34. After that, remember to create a *standard* configuration file:  
 ```
 # grub-mkconfig -o /boot/grub/grub.cfg
 ```
 35. As you can see, the directory structure of the `boot.efi` is not correct, as the `/System/Library/CoreServices` directory is not supposed to be a subdirectory of the `/boot/EFI/` folder. For this reason, we need to relocate the `boot.efi` stub in a location the [MacBook][mbp] bootloader is able to recognize:  
 ```
 # mv /boot/EFI/arch/System/ /boot/
 # rm -r /boot/EFI/ 
 ```
 36. After that, you need to create the following file `/boot/System/Library/CoreServices/SystemVersion.plist` if not already exists:  
 ```
 <?xml version="1.0" encoding="UTF-8"?>
 <plist version="1.0">
 <dict>
        <key>ProductBuildVersion</key>
        <string></string>
        <key>ProductName</key>
        <string>Linux</string>
        <key>ProductVersion</key>
        <string>Arch Linux</string>
 </dict>
 </plist>
 ```
 37. After the installation, it is optionally possible to set a custom icon that will be displayed in the [MacBook][mbp] boot loader. In order to do that, you need to install the `wget`, `librsvg` and `libicns` packages.  
 ```
 # pacman -S wget librsvg libicns
 ```
 38. After that, just follow the following commands:  
 ```
 # wget -O /tmp/archlinux.svg https://www.archlinux.org/logos/archlinux-icon-crystal-64.svg
 # rsvg-convert -w 128 -h 128 -o /tmp/archlogo.png /tmp/archlinux.svg
 # sudo png2icns /boot/.VolumeIcon.icns /tmp/archlogo.png
 # rm /tmp/archlogo.png
 # rm /tmp/archlinux.svg
 ```
 39. Exit from [Arch Linux][archlnx] CHROOT by:  
 ```
 # exit
 ```
 40. Unmount all by:  
 ```
 # umount -R /mnt
 ```
 41. Reboot and PRAY... :) The *Apple Boot Manager*, shown when holding down the **OPTION** key when booting the [MacBook][mbp], should display [Arch Linux][archlnx] as a possible boot option. Selecting that option will boot [GRUB][grub].  

Done! [GRUB][grub] can now be selected on the standard [MacBook][mbp] bootloader and you can boot into your newly installed [Arch Linux][archlnx].  

**NOTE**: For security reasons I suggest to lock the `ROOT` account by: `$ sudo passwd -l root` and DO NOT login as `ROOT`  

### License  

This text has been written by ©2020 DimiG

[archlnx]:https://www.archlinux.org
[archinst]:https://wiki.archlinux.org/index.php/installation_guide
[archmacinst]:https://wiki.archlinux.org/index.php/Mac
[lvm]:https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)
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
[pacman]:https://wiki.archlinux.org/index.php/pacman
[vim]:https://en.wikipedia.org/wiki/Vim_(text_editor)
[hfsplus]:https://en.wikipedia.org/wiki/HFS_Plus
[aur]:https://wiki.archlinux.org/index.php/Arch_User_Repository
[grub]:https://wiki.archlinux.org/index.php/GRUB
