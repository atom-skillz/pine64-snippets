# PineTab-V

## Everything related to the boot process

- Booting from eMMC or SD Card is no longer recommended, but it's the way the debian image created by the vendor is booting.

- SPL is in the first partition, U-BOOT is in the second partition

- U-Boot runs in Supervisor (S) Mode, SPL runs in Machine (M) Mode


## Initial disk layout

Disk /dev/mmcblk0: 116.47 GiB, 125057368064 bytes, 244252672 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: C11CD5A6-7841-4D8E-91C9-20FE42CC7B5F
First usable LBA: 34
Last usable LBA: 244252638
Alternative LBA: 244252671
Partition entries starting LBA: 2
Allocated partition entries: 128
Partition entries ending LBA: 33

Device          Start       End   Sectors Type-UUID                            UUID                                 Name Attrs
/dev/mmcblk0p1   4096      8191      4096 2E54B353-1271-4842-806F-E436D6AF6985 0A4EEBB2-A3F8-4500-8393-5C529A322B2D spl
/dev/mmcblk0p2   8192     16383      8192 5B193300-FC78-40CD-8002-E86C45580B47 664826F0-FE17-42C8-B3E9-7B32AAE67301 uboot
/dev/mmcblk0p3  16384    221183    204800 C12A7328-F81F-11D2-BA4B-00A0C93EC93B 0EE5B75B-6FC3-4C44-933D-989228A77479 image

Device          Start       End   Sectors   Size Type
/dev/mmcblk0p1   4096      8191      4096     2M HiFive BBL
/dev/mmcblk0p2   8192     16383      8192     4M HiFive FSBL
/dev/mmcblk0p3  16384    221183    204800   100M EFI System
/dev/mmcblk0p4 221184 219827404 219606221 104.7G Linux filesystem
/dev/mmcblk0p4 221184 219827404 219606221 0FC63DAF-8483-4772-8E79-3D69D8477DE4 99335532-AB3D-480C-8140-370D2ADA54B1 root Legae
