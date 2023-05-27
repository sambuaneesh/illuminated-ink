---
{"dg-publish":true,"permalink":"/arch-linux-install/"}
---


## Connecting Network
`iwctl` <--- open wifi settings
`device list` <--- show available wifi devices
`station YOURDEVICEHERE scan` <--- look for networks
`station YOURDEVICEHERE get-networks` <--- list available networks
`station YOURDEVICEHERE connect YOURNETWORKHERE` <--- connect to network
`quit` <--- leave wifi settings
`ping google.com` <-- to check internet connection working or not

---

## Fix Signing Keys issues
```bash
pacman-key --init
pacman-key --populate archlinux
```

---

## Verifying EFI or not
```bash
ls /sys/firmware/efi/efivars/
```
This should pop some results

---

## Update Date & Time
`timedatectl status` to check current status
`timedatectl list-timezones` to view available timezones
`timedatectl set-timezone Asia/Kolkata` set IST

---

## Partitioning
**Note** that the numbers and alphabets maybe varying in your case like /dev/sd**a** or /dev/sda**5**
### General Viewing
`lsblk` - to properly view the drives
`hdparam -i /dev/sda` - more info about the drive
`fdisk -l` - detailed drives view
### Create Partitions
`cfdisk /dev/sda` - to start making partitions
- Partitions
	- Root
	- Home
	- Swap (Min 4GB)
### Format Paritions
`mkfs.ext4 /dev/sda5` - to make ext4 partitions
`mkswap /dev/sda7` - to make swap partitions
`swapon /dev/sda7` - to mention swap mountpoint
Mounting Home and Root Partitions
```
mount /dev/sda5 /mnt
mkdir /mnt/home
mount /dev/sda6 /mnt/home
```

---

## Setting up Mirrors
`cp /etc/pacman.d/mirrorlist  /etc/pacman.d/mirrorlist.bak` Backing up current mirror list
`pacman -Sy` update pacman databases
`pacman -S pacman-contrib` install ranking tool
`rankmirrors -n 10 /etc/pacman.d/mirrorlist.bak > /etc/pacman.d/mirrorlist` rank the list and update the servers by their speed
you can view it again by using `cat` (`cat /etc/pacman.d/mirrorlist`)

---

## Install Arch to root partition
Installing necessary packages
```
pacstrap -i /mnt base base-devel linux linux-lts linux-headers linux-firmware intel-ucode sudo nano git neofetch networkmanager dhcpcd pulseaudio bluez wpa_supplicant
```

---

## Generate File System Table
When we boot to arch drive, we need to tell the system that we need to mount all the partitions under current boot `/mnt` to the same location
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
you can view it again by using `cat` (`cat /mnt/etc/fstab`)

---

## Entering Arch Drive
We can `chroot` to our installed arch drive `arch-chroot /mnt`
you can verify if you are there or not by typing `lsblk`
### Set Password and Sudo
`passwd` - set superuser password
you should not run system as a root user as a practice, so lets create a user
```bash
useradd -m stealthspectre
passwd stealthspectre
(and setup your password)
```
add to user groups `usermod -aG wheel,storage,power stealthspectre`
edit the sudoers file to access sudo
```bash
EDITOR=nano visudo
```
uncomment `%wheel ALL=...` this thing
you can add timeout for asking password below it by mentioning the time
`Defaults timestamp_timeout=0`
`Ctrl+O` - to write the changes
`Ctrl+X` - to exit the editor

---

## Setting System Language
`nano /etc/locale.gen`
scroll all the way and uncomment `en_US.UTF-8 UTF-8`
`locale-gen` - to generate the saved locale
`echo LANG=en_US.UTF8 > /etc/locale.conf` - create a locale config file
`export LANG=en_US.UTF-8` - export the system language

---

## Setup Host Name
```bash
echo ArchLinux > /etc/hostname
```
And add these lines to `nano /etc/hosts`
```md
127.0.0.1 <tabspace> localhost
::1 <tabspace>  localhost
127.0.1.1 <tabspace>  ArchLinux<this_is_your_hostname>.localdomain <tabspace>  localhost
```

---

## Setup TIme Zone and Local Time
```md
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
hwclock --systohc
```
you can use `<tab>` to see the timezones after `.../zoneinfo/`

---

## Setup GRUB
### Prerequisites
Constantly use `lsblk` to check whats happening and to select partitions
```bash
mkdir /boot/efi
mount /dev/sda1 /boot/efi/
pacman -S grub efibootmgr dosfstools mtools
```
### Edit and Updating GRUB config file
```bash
nano /etc/default/grub
```
uncomment `GRUB_DISABLE_OS_PROBER=false`
`Ctrl+O` - to write the changes
`Ctrl+X` - to exit the editor
```bash
pacman -S osprober
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Finalizing
`systemctl enable dhcpcd.service` Enabling network services (This service is responsible for providing the ip-address)
`systemctl enable NetworkManager.service`
Exit the chroot by `exit`
`umount -lR /mnt` - to unmount all the partitions
now run `systemctl daemon-reload` to make sure that grub is visible in the bootloader
`reboot` and remove the installation media

---