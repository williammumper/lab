START:  
Install arch linux iso on flash drive.  
Boot from USB on device

Select first option - install arch linux x86etc….

CLI:

itwctl #wifi

station wlan0 connect [wifi name]  
[wifi password]

exit

passwd #sets root password

systemctl start sshd #ssh into the install

systemctl enable sshd

ip a #Find inet (192.168.x.x), ssh in with ext device using ssh root@192.168.x.x and enter root password.

timedatectl #syncs the clock

lsblk #check the partitions

cfdisk /dev/_disk_to_partition_ #partition the disk - make one 1GB type EFI partition for /root/boot, put the rest to LVM

cryptsetup luksFormat /dev/mmcblk1p2

cryptsetup open /dev/mmcblk1p2 cryptlvm

pvcreate /dev/mapper/cryptlvm #create the encrypted physical volume

vgcreate lvm /dev/_LVM_partition_ #creates/names the volume group LVM

lvcreate -L 2G lvm -n swap #creates/names a 2Gb logical group under the lvm pv and vg for swap

lvcreate -l 100%FREE lvm -n root #creates/names the rest of the LVM storage to root.

mkfs.ext4 /dev/lvm/root #formats /root as ext4

mkswap /dev/lvm/swap #makes /swap fstype swap

mkfs.fat -F 32 /dev/_1GB_EFI_partition_

mount /dev/lvm/root /mnt

mount --mkdir /dev/mmcblk1p1 /mnt/boot

swapon /dev/lvm/swap

mkdir -p /mnt/etc

genfstab -U /mnt > /mnt/etc/fstab

pacstrap -K /mnt linux linux-firmware base

arch-chroot /mnt

ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime

sed -i 's/^#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen #uncomments UTF-8 in the /etc/locale.gen file

locale-gen

pacman -S vim

vim /etc/mkinitcpio.conf

paste into hooks:

HOOKS=(base **systemd** autodetect microcode modconf kms **keyboard** **sd-vconsole** block **sd-encrypt** **lvm2** filesystems fsck)

systemctl enable systemd-networkd.service

systemctl enable systemd-resolved.service

vim /etc/systemd/network/25-wireless.network

[Match]

Name=wlan0

[Network]

DHCP=yes

IgnoreCarrierLoss=3s

pacman -Syu iwd

pacman -Syu lvm2

vim /etc/vconsole.conf

FONT=latarcyrheb-sun16

bootctl --esp-path=/mnt/boot install #in root not in chroot

arch-chroot /mnt

bootctl install

bootctl update

vim /boot/loader/entries/arch.conf

paste:

title Arch Linux

linux /vmlinuz-linux

initrd /initramfs-linux.img

options rd.luks.name=3714390c-c870-4e74-b878-6c7f05d30c41=lvm root=/dev/lvm/root rw

should be UUID of device and the vg name

mkinitcpio -P

useradd will

usermod -aG wheel will

passwd will

pacman -Syu sudo

visudo

uncomment the #wheel one so wheel users can run everything

vim /boot/loader/loader.conf

default arch

vim /etc/hostname

[hostname]


checks

/boot mounted

/mnt mounted

/swap on

passwords set

user set and added to wheel

/etc/fstab has /boot

3rd try on this fucking install

i really should just write everything here

MAKE SURE TO CHECK FSTAB HAS /boot !!