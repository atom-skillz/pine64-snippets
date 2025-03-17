# PineTab2

## Encrypted root archlinux

Download the latest danctnix release from `https://github.com/dreemurrs-embedded/Pine64-Arch`

Uncompress then flash it to your sd card (/dev/sda in this case)
```
xz -d archlinux-pinetab2-barebone-20241223.img.xz
dd if=archlinux-pinetab2-barebone-20241223.img of=/dev/sda
```

Mount the root partition and copy it to the root partition
```
mount /dev/sda2 /mnt
cp archlinux-pinetab2-barebone-20241223.img /mnt
umount /dev/sda2
```

Insert sd card into your PineTab2 and boot the new image. Update the system time, then update the system and reboot so the necessary packages can be installed in the temporary system.
```
nmtui   # Set up networking
pacman -Syu

# Installing necessary packages
pacman -S arch-install-scripts
```

Use `losetup` to set the image for mounting, that way we can use the individual partitions.
```
losetup -P loop0 archlinux-pinetab2-barebone-20241223.img
```

Create a GPT partition table in the eMMC with the following characteristics:
- Boot partition, start at 62500 (EFI, at least 125M)
- Encrypted partition (Default, rest of disk)

From that, you should be able to encrypt that partition and set up the encrypted area as an ext4: 
```
cryptsetup luksFormat /dev/mmcblk0p2
cryptsetup luksOpen /dev/mmcblk0p2 root
mkfs.ext4 /dev/mapper/root  # Could be any other
```

Set up the necessary folders and mount the original image and the new partition:
```
mkdir /mnt/image
mount /dev/loop0p2 /mnt/image

mkdir /mnt/new_root
mount /dev/mapper/root /mnt/new_root
```

Copy the contents to the new partition:
```
cp -vax /mnt/image/* /mnt/new_root
sync
```

Copy the boot partition into the disk, then mount it as `/boot`:
```
dd if=/dev/loop0p1 of=/dev/mmcblk0p1 status=progress
mount /dev/mmcblk0p1 /mnt/new_root/boot
```

Chroot into the new root for extra tweaking and updating the base system:
```
arch-chroot /mnt/new_root

pacman-key --init
pacman-key --populate danctnix
pacman-key --populate archlinuxarm
pacman -Syu
```

Edit your `/etc/fstab` to update the UUID of the root partition:
```
blkid /dev/mapper/root
nano /etc/fstab
```

Edit the initramfs settings up add the `encrypt` hook:
```
nano /etc/mkinitcpio.conf
mkinitcpio -P
```

Update your `boot.txt` script so that the encrypted partition is initialized with `cryptdevice=` and `root=` points to the new root system:

```
[...]
setenv bootargs loglevel=4 cryptdevice=/dev/mmcblk${linux_mmcdev}p${rootpart}:root root=/dev/mapper/root rw rootwait
[...]
```

Update uboot:
```
cd /boot
./mkscr

dd if=/boot/idbloader.img of=/dev/mmcblk0 bs=512 seek=64
dd if=/boot/u-boot.itb of=/dev/mmcblk0 bs=512 seek=16384
```

With this, your root partition will be encrypted and your root password will be obtained via the UART console. This is not very practical, but it is still useable... somewhat.


## Adding USB unlock

Source:
`https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Unlocking_the_root_partition_at_boot`

Automating the boot process requires generating a usb key for the password. For that, we need to first generate a secret key for that USB device:
```
dd if=/dev/urandom of=secret-key.lek bs=1 count=8192 iflag=fullblock
cryptsetup luksAddKey /dev/mmcblk0p2 secret-key.lek
```

Add `ext4` module to the initramfs:
```
[...]
MODULES=(ext4)
[...]
```

Create the disk you are going to be using for the keyfile (/dev/sda1 in this case), then add the keyfile:
```
mkinitcpio -P

mkfs.ext4 /dev/sda1
mount /dev/sda1 /mnt
cp secret-key.lek /mnt
```

That way we can add the following snippet to our `boot.txt`
```
cryptkey=/dev/sda1:ext4:/secret-key.lek
```

Update uboot the same way as before and reboot. It should be ready now.
