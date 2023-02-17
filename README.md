# Installation guide

This installation guide is bases on the official [Arch Linux](https://wiki.archlinux.org/title/Installation_guide) page.

[![arch](https://img.shields.io/badge/arch_linux-blue?style=for-the-badge&logo=archlinux&logoColor=white&logoWidth=20)](https://archlinux.org/)

## Keyboard layout

To set the keyboard layout:

```bash
loadkeys la-latin1
```

Available layouts can be listed with:

```bash
ls /usr/share/kbd/keymaps/**/*.map.gz
```

## Booted mode

To verify the boot mode, list the efivars directory:

```bash
ls /sys/firmware/efi/efivars
```

If the command shows the directory without error, then the system is booted in UEFI mode. If the directory does not exist, the system may be booted in BIOS mode.

## Connect to the internet

It is recommended to coccect to the interner through an Ethernet cable.

## System clock

Use **timedatectl** to ensure the system clock is accurate:

```bash
timedatectl status
```
## Partition the disks

To identify the devices, use **lsblk**. 

Use **cfdisk** to modify partition tables:

```bash
cfdisk /dev/the_disk_to_be_partitiones
```

|Mount point    |Type       |size                     |
|---------------|-----------|-------------------------|
|/mnt/boot/efi  |Efi system |1 GB                     |
|[SWAP]         |Linux swap |More than 512 MB         |
|/mnt           |Root(/)    |Desired size             |
|/mnt/home      |Home       |Remainder of the device  |

## Format the partitions

Format the partitions with the proper file system.

```bash
mkfs.ext4 -L ROOT /dev/root_partition
mkfs.ext4 -L HOME /dev/home_partition
mkswap -L SWAP /dev/swap_partition
mkfs.fat -F 32 /dev/efi_partition
fatlabel /dev/efi_partition BOOT
```

## Mount the file system

To mount the partitions created is the following.

```bash
mount /dev/root_partition /mnt
mount --mkdir /dev/efi_partition /mnt/boot/efi
mount --mkdir /dev/home_partition /mnt/home
swapon /dev/swap_partition
```

## Install essential package

Use the pacstrap script to install the base package, Linux kernel and firmware for common hardware:

```bash
pacstrap -K /base linux-zen linux-firmware
```

You can add other packages you need or use pacman while chrooted into the new systm.

Packages I recommendes installing.

```bash
neovim networkmanager grub os-prober efibootmgr
```

## Fstab

Generate an fstab file.

```bash
genfstab -L /mnt >> /mnt/etc/fstab
```

## Chroot

Change root into the new system:

```bash
arch-chroot /mnt
```

## Time zone

Set the time zone:

```bash
ls -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Run **hwclock** to generate _/etc/edjtime_:

```bash
hwclock --systohc
```

## Localization

Edit _/etc/locale.gen_. Generate the locales by running.

```bash
locale-gen
```

Create the **locale.conf** file and **vconsole.conf**.

```bash
echo "LANG=es_MX.UTF-8" >> /etc/locale.conf
echo "KEYMAP=la-latin1" >> /etc/vconsole.conf
```

## Network configuration

Create the hostname file.

```bash
echo "hostname" >> /etc/hostname
```

Enable NetworkManager services if the package is installed.

```bash
pacman -S networkmanager

systemctl enable NetworkManager.services
```

## Root password

Set the root password

```bash
passwd
```

## Boot loader

Install the GRUB bootloader

```bash
pacman -S grub efibootmgr os-prober
```

Execute the following command to install the GRUB EFI.

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

Add the following to _/etc/default/grub. then re-run grub-mkconfig.

```bash
GRUB_DISABLE_OS_PROBER=false
```






