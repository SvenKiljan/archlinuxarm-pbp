# Installation

Arch Linux ARM can be installed on and booted from a microSD card, an eMMC module or an NVME SSD.

## Installation on microSD card or eMMC module

Replace sdX in the following instructions with the device name for the microSD card or eMMC module as it appears on your computer.

1. Zero the beginning of the SD card or eMMC module:

```
dd if=/dev/zero of=/dev/sdX bs=1M count=32
```

2. Start fdisk to partition the SD card or eMMC module:

```
fdisk /dev/sdX
```

3. At the fdisk prompt, create the new partition:

   a. Type **o**. This will clear out any partitions on the drive.

   b. Type **p** to list partitions. There should be no partitions left.

   c. Type **n**, then **p** for primary, **1** for the first partition on the drive, **32768** for the first sector, and then type **442367** for the last sector.

   d. Type **t**, then **c** to set the first partition to type W95 FAT32 (LBA).

   e. Type **n**, then **p** for primary, **2** for the second partition on the drive, **442368** for the first sector, and then press ENTER to accept the default last sector. 

   f. Write the partition table and exit by typing **w**.

4. Create and mount the FAT filesystem:

```
mkfs.vfat -n BOOT_ALARM /dev/sdX1
mkdir boot
mount /dev/sdX1 boot
```
  
5. Create and mount the ext4 filesystem:

```
mkfs.ext4 -L ROOT_ALARM /dev/sdX2
mkdir root
mount /dev/sdX2 root
```

6. Download and extract the root filesystem (as root, not via sudo):

```
wget https://github.com/SvenKiljan/archlinuxarm-pbp/releases/latest/download/ArchLinuxARM-pbp-latest.tar.gz
bsdtar -xpf ArchLinuxARM-pbp-latest.tar.gz -C root
```

7. Move boot files to the first partition: 

```
mv root/boot/* boot
```

8. Install the Tow-Boot bootloader:

```
dd if=boot/idbloader.img of=/dev/sdX seek=64 conv=notrunc,fsync
dd if=boot/tow-boot.itb of=/dev/sdX seek=16384 conv=notrunc,fsync
```

9. Unmount the two partitions:

```
umount boot root
```

11. Insert the SD card or eMMC module into the Pinebook Pro, and power it on.

12. After logging in as root (password is "root"), you can connect to a wireless network by running:

```
wifi-menu
```

13. Synchronize the system and RTC clocks:

```
timedatectl set-ntp on
hwclock -w
```

14. Initialize the pacman keyring and populate the Arch Linux ARM and Pinebook Pro [package signing](https://archlinuxarm.org/about/package-signing) keys:

```
pacman-key --init
pacman-key --populate archlinuxarm
pacman-key --populate archlinuxarm-pbp
```


## Installation on eMMC module without a USB adapter

1. Install, boot and configure Arch Linux ARM on a microSD card first by following [these instructions](#installation-on-microsd-card-or-emmc-module).

2. Zero the beginning of the eMMC module:

```
dd if=/dev/zero of=/dev/mmcblk2 bs=1M count=32
```

3. Start fdisk to partition the SD card or eMMC module:

```
fdisk /dev/mmcblk2
```

3. At the fdisk prompt, create the new partition:

   a. Type **o**. This will clear out any partitions on the drive.

   b. Type **p** to list partitions. There should be no partitions left.

   c. Type **n**, then **p** for primary, **1** for the first partition on the drive, **32768** for the first sector, and then type **442367** for the last sector.

   d. Type **t**, then **c** to set the first partition to type W95 FAT32 (LBA).

   e. Type **n**, then **p** for primary, **2** for the second partition on the drive, **442368** for the first sector, and then press ENTER to accept the default last sector. 

   f. Write the partition table and exit by typing **w**.

4. Create and mount the ext4 filesystem:

```
mkfs.ext4 -L ROOT_ALARM /dev/mmcblk2p2
mount /dev/mmcblk2p2 /mnt
```

5. Create and mount the FAT filesystem:

```
mkfs.vfat -n BOOT_ALARM /dev/mmcblk2p1
mkdir -p /mnt/boot
mount /dev/mmcblk2p1 /mnt/boot
```
 
6. Transfer all data from the microSD card to the eMMC module:

```
rsync -aAXq --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/lost+found"} / /mnt
```

7. Install the Tow-Boot bootloader:

```
dd if=/mnt/boot/idbloader.img of=/dev/mmcblk2 seek=64 conv=notrunc,fsync
dd if=/mnt/boot/tow-boot.itb of=/dev/mmcblk2 seek=16384 conv=notrunc,fsync
```

8. Unmount the two partitions:

```
umount -R /mnt
```

9. Power off the Pinebook Pro, remove the SD card, and power it on.


## Installation on NVME

1. Install, boot and configure Arch Linux ARM on a microSD card first by following [these instructions](#installation-on-microsd-card-or-emmc-module).

2. Zero the beginning of the NVME SSD:

```
dd if=/dev/zero of=/dev/nvme0n1 bs=1M count=32
```

3. Start fdisk to partition the NVME SSD:

```
fdisk /dev/nvme0n1
```

3. At the fdisk prompt, create the new partition:

   a. Type **o**. This will clear out any partitions on the drive.

   b. Type **p** to list partitions. There should be no partitions left.

   c. Type **n**, then **p** for primary, **1** for the first partition on the drive, **32768** for the first sector, and then type **442367** for the last sector.

   d. Type **t**, then **c** to set the first partition to type W95 FAT32 (LBA).

   e. Type **n**, then **p** for primary, **2** for the second partition on the drive, **442368** for the first sector, and then press ENTER to accept the default last sector. 

   f. Write the partition table and exit by typing **w**.

4. Create and mount the ext4 filesystem:

```
mkfs.ext4 -L ROOT_ALARM /dev/nvme0n1p2
mount /dev/nvme0n1p2 /mnt
```

5. Create and mount the FAT filesystem:

```
mkfs.vfat -n BOOT_ALARM /dev/nvme0n1p1
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```
 
6. Transfer all data from the microSD card to the NVME SSD:

```
rsync -aAXq --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/lost+found"} / /mnt
```

7. Install the Tow-Boot bootloader in SPI flash:

```
flash_erase /dev/mtd0 0 0
nandwrite -p /dev/mtd0 /mnt/boot/tow-boot.spiflash.bin
```

8. Unmount the two partitions:

```
umount -R /mnt
```

9. Power off the Pinebook Pro, remove the SD card, and power it on.
