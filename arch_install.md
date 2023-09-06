# ArchLinux install notes

## Check if UEFI is enabled

    ls /sys/firmware/efi/efivars

## Connect to WiFi

    iwctl
    [iwd]# device list
    [iwd]# station device scan
    [iwd]# station device get-networks
    [iwd]# station device connect SSID

## Check connection

    ping archlinux.org

## Update the system clock

    timedatectl set-ntp true

## Partition the disks

    cfdisk /dev/disk

### Example layouts

#### Non-UEFI

| Mount point | Partition | Partition type | Suggested size |
| --- | --- | --- | --- |
| /mnt | /dev/root_partition | Linux filesystem | 20GB |
| [SWAP] | /dev/swap_partition | Linux swap | Double system RAM |

#### UEFI

| Mount point | Partition | Partition type | Suggested size |
| --- | --- | --- | --- |
| /mnt/boot or /mnt/efi | /dev/efi_system_partition | EFI system partition | 512MB |
| /mnt | /dev/root_partition | Linux filesystem | 20GB |
| [SWAP] | /dev/swap_partition | Linux swap | Double system RAM |

### Formart partition

#### Non-UEFI

    mkfs.ext4 /dev/root_partition
    mkswap /dev/swap_partition
    swapon /dev/swap_partition

#### UEFI

    mkfs.fat -F32 /dev/efi_system_partition
    mkfs.ext4 /dev/root_partition
    mkswap /dev/swap_partition
    swapon /dev/swap_partition

## Mount filesystem

    mount /dev/root_partition /mnt

## Installation

### Update mirrors

    mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
    curl -Lo /etc/pacman.d/mirrorlist 'https://www.archlinux.org/mirrorlist/?country=AT&country=CZ&country=SK&protocol=http&protocol=https&ip_version=4'

### Install packages

    pacstrap /mnt base base-devel linux linux-firmware man-db man-pages texinfo neovim sudo networkmanager zsh ansible git

## Configure system

### Generate fstab

    genfstab -U /mnt >> /mnt/etc/fstab

### Chroot

    arch-chroot /mnt

### Set timezone and hwclock

    ln -sf /usr/share/zoneinfo/Europe/Bratislava /etc/localtime
    hwclock --systohc

### Localization

#### Generate locales

    nvim /etc/locale.gen
    --------------------
    Uncomment desired locales

    locale-gen

#### Set system-widge locale

    echo "LANG=en_US.UTF-8" > /etc/locale.conf

### Network configuration

#### Set hostname

    echo YourPCHostName > /etc/hostname

#### Edit hosts

    nvim /etc/hosts
    ----------------
    127.0.0.1 localhost
    ::1 localhost
    127.0.1.1 myhostname.localdomain myhostname
    
#### Set global DNS

    nano /etc/NetworkManager/conf.d/dns-servers.conf

    [global-dns-domain-*]
    servers=1.1.1.1,1.0.0.1,208.67.222.222,208.67.220.220
    
    systemctl enable NetworkManager

### Set root password

    passwd

## Bootloader

### Non-UEFI

    pacman -S grub
    grub-install --target=i386-pc /dev/disk
    grub-mkconfig -o /boot/grub/grub.cfg

### UEFI

    pacman -S grub efibootmgr
    mkdir /boot/efi
    mount /dev/sda1 /boot/efi
    lsblk # to check if everything is mounted correctly
    grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi --removable
    grub-mkconfig -o /boot/grub/grub.cfg

## Finnish

    exit
    umount -R /mnt
    reboot
