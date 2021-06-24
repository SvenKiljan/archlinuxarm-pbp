# Arch Linux ARM for Pinebook Pro

This repository provides root filesystems and instructions for installing and running [Arch Linux ARM](https://archlinuxarm.org/) on [Pinebook Pro](https://www.pine64.org/pinebook-pro/) laptops.

Installation instructions can be found here: [INSTALL.md](INSTALL.md)

Root filesystems can be found here: [Releases](https://github.com/SvenKiljan/archlinuxarm-pbp/releases)

If you have any issues or questions, read this first: [FAQ.md](FAQ.md)

## About Pinebook Pro

The [Pinebook Pro](https://www.pine64.org/pinebook-pro/) is a laptop powered by an ARMv8 Rockchip RK3399 hexa-core processor and 4GB RAM, measuring 329 mm x 220 mm x 12 mm and weighing 1.26 kg.

Features

- Rockchip RK3399 dual-core 2.0GHz Cortex-A72 and quad-core 1.5GHz Cortex-A53 processor
- 4GB LPDDR4 RAM
- 14" 1080p color active matrix TFT LCD
- Mali T860MP4 GPU
- 16MB SPI NAND flash storage (bootable)
- 64GB or 128GB eMMC module (user replaceable, bootable)
- microSD reader (bootable)
- M.2/NGFF NVMe SSD slot (requires [a separate adapter](https://pine64.com/product/pinebook-pro-m-2-ngff-nvme-ssd-interface-adapter/?v=0446c16e2e66))
- 10,000 mAH battery
- 1x USB 3.0 Type-C port
- 1x USB 3.0 Type-A port
- 1x USB 2.0 Type-A port

## About Arch Linux ARM on the Pinebook Pro

[Arch Linux ARM](https://archlinuxarm.org/) does not officially support the Pinebook Pro. This repository provides a root filesystem based on Arch Linux ARM with some [additional packages](https://pacman.kiljan.org/archlinuxarm-pbp/os/aarch64/). The PKGBUILDs of these additional packages can be found [here](https://github.com/SvenKiljan/archlinuxarm-pbp-packages).

Manjaro ARM is the semi-official distribution of the Pinebook Pro. The device ships with it, and it seems most development for this particular device ends up in this distribution. The additional Arch Linux ARM packages to support the Pinebook Pro are mostly based on those offered by Manjaro ARM, with some minor adjustments to make them fit better in the Arch Linux ARM ecosystem.

What works:

Function | Status | Remark
--- | --- | ---
**Basics**
Boot Manjaro ARM kernel | ✔ |
**Human interfacing**
Internal display | ✔ |
Keyboard | ✔ |
Touchpad | ✔ |https://github.com/SvenKiljan/archlinuxarm-pbp/blob/main/FAQ.md#why-is-direct-nvme-ssd-booting-considered-buggy
**Storage**
SPI | ✔ | Only usable for the bootloader. Optional to use.
eMMC module | ✔ | As bootable system disk and regular disk.
microSD reader | ✔ | As bootable system disk and regular disk. See the [compatibility list](https://wiki.pine64.org/wiki/Pinebook_Pro_Hardware_Accessory_Compatibility#microSD_Cards).
NVMe SSD | ✔* | * [Experimenta](https://wiki.pine64.org/wiki/Pinebook_Pro_Troubleshooting_Guide#NVMe_SSD_issues). As system disk and regular disk. Not directly bootable. Can be made bootable using an eMMC module (included with the notebook) or by executing the bootloader from SPI ([not recommended](https://github.com/SvenKiljan/archlinuxarm-pbp/blob/main/FAQ.md#why-is-direct-nvme-ssd-booting-considered-buggy)). Incompatible with Suspend to RAM. [Needs an adapter](https://pine64.com/product/pinebook-pro-m-2-ngff-nvme-ssd-interface-adapter/). See the [compatibility list](https://wiki.pine64.org/wiki/Pinebook_Pro_Hardware_Accessory_Compatibility#NVMe_SSD_drives).
USB Mass Storage Devie | ✔* | * As system disk and regular disk. Not directly bootable. Can be made bootable by executing the bootloader from SPI.
**Other internal hardware**
Wireless network adapter | ✔ |
Bluetooth | ✔ |
Audio | ✔ | Including headphone detection.
Webcam | ✔ |
**Expansion ports**
USB | ✔ | USB-C 3.0, USB-A 3.0 and USB-A 2.0.
External display (using USB-C DisplayPort Alternate Mode) | ✔* | * Requires a [compatible USB-C dock](https://wiki.pine64.org/wiki/Pinebook_Pro_Hardware_Accessory_Compatibility#USB_C_alternate_mode_DP). Tested with HDMI and DVI-D monitors. Video works, [audio does not](https://forum.manjaro.org/t/no-hdmi-audio-on-pinebook-pro/50203/2).
**Energy management** | |
Lid close detection | ✔ | Older Pinebook Pro laptops might have a misaligned magnet, [which can be fixed](https://wiki.pine64.org/wiki/Pinebook_Pro_Troubleshooting_Guide#Pinebook_Pro_will_not_sleep_with_lid_closed).
Power down and reboot | ✔ |
Suspend to Idle | ✔ | Consumes around 5% battery capacity every hour.
Suspend to RAM (S3) | ✔* | * Experimental and incompatible with NVME. See [this FAQ entry](FAQ.md#why-does-the-system-consume-so-much-energy-when-sleeping).
Suspend to disk (S4) | ❌ | Does not work for any distribution at this time. To compensate somewhat for this shortcoming, consider using a desktop environment that can restore a previous session, such as Plasma.
**Other** |  | 
Video acceleration | ✔ | For YouTube playback in browsers, check the [FAQ](FAQ.md#why-is-youtube-so-slow).

## More information

* [Pinebook Pro on the PINE64 wiki](https://wiki.pine64.org/index.php/Pinebook_Pro)
* [Arch Linux ARM topic in the PINE64 Pinebook Pro forum](https://forum.pine64.org/showthread.php?tid=14238)

## Acknowledgments

* [Nadia Holmquist Pedersen](https://nhp.sh/) for laying the groundwork and for providing solutions to several issues.
* All the people working on [Manjaro ARM](https://manjaro.org/), especially the contributors in the [PKGBUILDs](https://pacman.kiljan.org/archlinuxarm-pbp/).
* All the people working on [Tow-Boot](https://github.com/Tow-Boot/Tow-Boot), for providing a Pinebook Pro boot loader that supports Suspend to RAM and SPI booting.
* All the people working on [Arch Linux ARM](https://archlinuxarm.org/).
* All authors of the [ArchWiki](https://wiki.archlinux.org/).
* [PINE64](https://www.pine64.org/), for providing affordable hardware with a prime focus on community-driven development.
