# Installation

## Create a microSD or eMMC boot medium

Replace sdX in the following instructions with the device name for the microSD card or eMMC module as it appears on your computer.

1. Zero the beginning of the SD card or eMMC module:

```
dd if=/dev/zero of=/dev/sdX bs=1M count=32
```

2. Start fdisk to partition the SD card:

```
fdisk /dev/sdX
```

3. At the fdisk prompt, create the new partition:

   a. Type **o**. This will clear out any partitions on the drive.

   b. Type **p** to list partitions. There should be no partitions left.

   c. Type **n**, then **p** for primary, **1** for the first partition on the drive, **32768** for the first sector, and then type **442367** for the last sector.

   d. Type **t**, then **c** to set the first partition to type W95 FAT32 (LBA).

   e. Type **n**, then **p** for primary, **2** for the second partition on the drive, **442368** for the first sector, and then press ENTER to accept the default last sector. 

   f. Write the partition table and exit by typing w.

4. Create and mount the FAT filesystem:

```
mkfs.vfat -n BOOT_ALARM /dev/sdX1
mkdir boot
mount /dev/sdX1 boot
```
  
5. Create and mount the ext4 filesystem:

```
mkfs.ext4 -L ROOT_ALARM /dev/sdX1
mkdir root
mount /dev/sdX2 root
```

6. Download and extract the root filesystem (as root, not via sudo):

```
wget https://os.kiljan.org/os/ArchLinuxARM-pbp-latest.tar.gz
bsdtar -xpf ArchLinuxARM-pbp-latest.tar.gz -C root
```

7. Move boot files to the first partition: 

```
mv root/boot/* boot
```

8. Install the U-Boot bootloader:

```
dd if=boot/idbloader.img of=/dev/sdX seek=64 conv=notrunc,fsync
dd if=boot/u-boot.itb of=/dev/sdX seek=16384 conv=notrunc,fsync
```

9. Reconfigure the U-Boot bootloader if the medium is an eMMC module (skip this step if a microSD card is used):

```
sed -i 's/mmcblk1/mmcblk2/' root/boot/extlinux/extlinux.conf
```

10. Unmount the two partitions:

```
umount boot root
```

11. Insert the SD card or eMMC module into the Pinebook Pro, and power it on.

12. After logging in as root (password is "root"), you can connect to a wireless network by running:

```
wifi-menu
```

12. After logging in as root (password is "root"), you can connect to a wireless network by running:

13. Initialize the pacman keyring and populate the Arch Linux ARM and Pinebook Pro [package signing](https://archlinuxarm.org/about/package-signing) keys:

```
pacman-key --init
pacman-key --populate archlinuxarm
pacman-key --populate archlinuxarm-pbp
```
