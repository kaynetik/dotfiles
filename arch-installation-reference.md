# Arch Install

## Init Network

Check if it was resolved automatically

```bash
ping -c 3 archlinux.org
```

If not, resolve it:
Wired   `ip link`
Wi-Fi   `wifi-menu`

## Partritioning

```bash
fdisk -l
cfdisk /dev/sda
```

### Recommended scheme (without swap)

```bash
    * /dev/sda1     # 400 MB (UEFI)
    * /dev/sda2     # 50 GB (root)
    * /dev/sda3     # XYZ (home)
```

If you choose to go with cfdisk, mark `sda1` as EFI boot part flag, and other two as `Linux file system` (if I remember correctly).

### Format partitions

```bash
mkfs.fat -F31 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3
```

## Mount partitions

```bash
mount /dev/sda2 /mnt
mkdir /mnt/home
mount /dev/sda3 /mnt/home
```

Double check that everything is mounted as expected with `lsblk`. Note that there was no need to mount EFI part at this moment.

## Install Arch

```bash
pacstrap -i /mnt base base-devel
```

Select _all_ and _yes_.

## Generate fstab

```bash
genfstab -U -p /mnt >> /mnt/etc/fstab
```

## Chroot into fresh & dandy Archy

```bash
arch-chroot /mnt /bin/bash
sudo pacman -S vim
```

## Set locale

### Time & Language

```bash
vim /etc/locale.gen
locale-gen

ln -sf /usr/share/zoneinfo/Europe/Warsaw /etc/localtime
hwclock --systohc --utc
```

### Host

```bash
echo Archy > /etc/hostname

vim /etc/hosts
    > 127.0.1.1  localhost.localdomain Archy
```

## Network

```bash
pacman -S networkmanager

systemctl enable NetworkManager
```

## Bootloader

### Set root password

```bash
passwd
```

### Install GRUB

```bash
pacman -S grub efibootmgr

mkdir /boot/efi
mount /dev/sda1 /boot/efi

lsblk       # check all is fine

grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

## Reboot

```bash
umount -R /mnt
reboot -f
```

## Add User

Login as root, and add a regular user

```bash
useradd -m -g users -G wheel -s /bin/bash USERNAME

passwd USERNAME
```

### Enable sudo for this user

```bash
EDITOR=vim visudo
```

Using `:/<pattarn>`, find `# %wheel ALL=(ALL) ALL`, and uncomment it.

## Install Audion, DMS, DM and WM of your choosing

```bash
pacman -S pulseaudio pulseaudio-alsa xorg xorg-xinit xorg-server
```

### Xfce4

```bash
pacman -S xfce4 lightdm lightdm-gtk-greeter
echo "exec startxfce4" > ~/.xinitrc

systemctl enable lightdm

startx
```

### Gnome3

```bash
pacman -S gnome
echo "exec gnome-session" > ~/.xinitrc

sudo systemctl enable gdm.service
systemctl start gdm.service

startx
```

### i3 (recommended)

```bash
pacman -S i3        # accept all recommended packages
echo "exec i3" > ~/.xinitrc
```
