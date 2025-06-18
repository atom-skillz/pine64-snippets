# PineTab 2

## Booting Fedora 42 workstation release

Download the raw image from the latest Fedora release from `https://fedoraproject.org/workstation/download/`

Uncompress the image and mount for `losetup`:
```
xz -d Fedora-Workstation-Disk-42-1.1.aarch64.raw.xz
sudo losetup -f -P Fedora-Workstation-Disk-42-1.1.aarch64.raw
```

The image must be mounted with `-P` so we can access the partitions inside. There will be 3 partitions: 
- A 500MB EFI system partition
- A 1GB ext4 boot partition
- A 10GB btrfs root partition

The idea of this guide is to move the partitions into a new structure that supports the PineTab2 using the u-boot in the danctnix repositories.


Note: Incomplete, needs kernel .spec file for danct kernel
