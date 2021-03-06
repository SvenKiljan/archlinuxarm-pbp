# Frequently asked questions

## Which devices are supported?

This repository goal is to make Arch Linux ARM installable and usable on the PINE64 Pinebook Pro. Other (Roockhip-based) devices might work [with some minor adjustments](https://forum.pine64.org/showthread.php?tid=14238&pid=97215#pid97215), but be prepared to work around any unforeseen complications.

## How do I migrate from other Arch Linux ARM releases for the Pinebook Pro?

This has not been tested due to that the old repository was suddenly not available anymore. Take care, and make backups in case a reinstallation is necessary.

First, open and examine `/etc/pacman.conf`. See if the following section exists:
```
[pinebookpro]
Server = https://nhp.sh/pinebookpro/
```

If it exists, read on. If not, adjust the instructions accordingly.

Remove installed packages from custom repository:

```
pacman -R $(for customPackage in $(pacman -Sl pinebookpro | awk '/installed.$/{print $2;}'); do customPackages="$customPackages $customPackage"; done && echo $customPackages)
```

Edit `/etc/pacman.conf` and remove the custom repository.

Now run:

```
pacman-key --keyserver hkps://keys.openpgp.org/ --recv-keys A1EC3C686EF7A9DD232D1563D4D12D6AA6A92769
pacman-key --lsign-key A1EC3C686EF7A9DD232D1563D4D12D6AA6A92769

cat << 'EOF' >> /etc/pacman.conf

[archlinuxarm-pbp]
SigLevel = Optional TrustedOnly
Server = http://pacman.kiljan.org/$repo/os/$arch
EOF
```

Update Arch Linux ARM and install the recommended packages from the archlinuxarm-pbp repository:
```
pacman -Syu ap6256-firmware libdrm-pinebookpro linux-manjaro pinebookpro-audio pinebookpro-post-install towboot-pinebookpro-bin
```

If all went well, everything was replaced except the boot loader and its configuration (located in `/boot/extlinux/extlinux.conf`). The existing boot loader should be able to boot the new kernel.

It might be that [/etc/fstab needs to be adjusted](https://forum.pine64.org/showthread.php?tid=14238&pid=96807#pid96807), by commenting the line referring to /boot. This is necessary if Arch Linux ARM was installed on a single partition previously.


## Why is the version of the kernel named 'MANJARO-ARM'?

Manjaro ARM is the de facto operating system of the Pinebook Pro since it is installed on newer Pinebook Pro laptops by default, and as close to 'upstream' as we can get for an Arch Linux-based distribution. If improvements are made for the Pinebook Pro in the Linux kernel, it is likely that Manjaro ARM will implement them first. A while ago, Manjaro ARM included Pinebook Pro support in their mainline kernel and deprecated the Pinebook Pro specific kernel. This is why the mainline Manjaro ARM kernel is used for Arch Linux ARM. The name was kept in the version to make it clear that any bugs or improvements should be reported upstream.

The Manjaro ARM kernel used in Arch Linux ARM for Pinebook Pro is built on an Arch Linux ARM system as a pacman package. Combined with how close Manjaro ARM is to Arch Linux (ARM), its installation and use should not provide any compatibility issues.

## Why is direct NVMe SSD booting considered buggy?

While testing with a Crucial P1 SSD, initially direct booting (with the bootloader in SPI) worked fine. However, after several reinstallations using the same software, the Linux kernel was unable to initialize the NVMe SSD correctly if the bootloader used it to load files from. [This problem is also experienced by others](https://forum.pine64.org/showthread.php?tid=11699&pid=94165#pid94165). A workaround is to re-bind the PCI device to the NVMe driver, but that can only be done after the system has booted (chicken and egg problem). Another [workaround described in ArchWiki](https://wiki.archlinux.org/title/Solid_state_drive/NVMe#Controller_failure_due_to_broken_APST_support) had no effect.

An alternative that 'just works' is a staged boot, in which the bootloader and the boot partition are loaded from the the eMMC module. This requires zero configuration after installation, and brings all the advantages of using NVMe. As a bonus, the SPI flash does not need to be written or used, since the ROM code of the Pinebook Pro can load the bootloader directly from eMMC.

The [installation instructions](https://github.com/SvenKiljan/archlinuxarm-pbp/blob/main/INSTALL.md#installation-on-nvme-ssd-staged-boot-using-emmc-module) for a staged eMMC+NVMe boot reserve space on the NVMe SSD. This will allow a boot partition to be created later, if this issue is ever solved in the bootloader and/or in Linux.


## Why do secure connections fail, such as when updating the Pinebook Pro?

Check the date and time with `timedatectl`. If the local time, the universal (system) time or RTC (hardware) time significantly differ from the current time, this is the likely cause of the connection problems.

To set the local timezone, first see what the name is of the local timezone based on the continent and closest city (exit with CTRL-C or :q):

```
timedatectl list-timezones
```

Now set the local timezone by running as root (replace Content/City with the value from the list given by the previous command):
```
timedatectl set-timezone Continent/City
```

To automatically synchronize the system time with online NTP servers, run as root:

```
timedatectl set-ntp on
```

To synchronize the hardware time using the system time, run as root:

```
hwclock --systohc
```

More information can be found on the [ArchWiki](https://wiki.archlinux.org/title/System_time).


## Why does the battery discharge when running with an external charger?

This happens when an inadequate power supply is used.

The power circuit of the Pinebook Pro can only use up to 15 watt (5V 3A). The power has to be shared between running the notebook and charging the battery. When using the power supply included with the Pinebook Pro, the battery will charge even when under heavy load.

Some tests were done to see how this works. For the tests in which the battery was charging, the current charge was around 50-60%. For the tests for in which display brightness is not explicitly defined, it was at maximum brightness.

An AC power meter was used to measure the power. The device used is not very accurate, but good enough to get some indications.

DC = included power supply (5V 3A) using the DC connector.
USB = USB power supply using the USB C connector, tested with an official Raspberry Pi 4 power supply (5.1V 3A)
USB PD = USB Power Delivery power supply using the USB C connector, tested with a power supply rated for 5V 3A and 5.2V 2.4A.


Connection | Power state | Battery state | Consumed power | Remarks
--- | --- | --- | --- | ---
DC | Off | Charging | 12-13 watt |
DC | On  | Charging | 15 watt | Idle, display at minimal brightness
DC | On  | Charging | 15 watt | Idle, display at maximum brightness
USB | Off | Charging | 12 watt | Idle
USB | On | Charging | 12 watt | Idle
USB PD | Off | Charging | 12 watt | Idle
USB PD | On | Charging | 12 watt | Idle
| | | |
DC | Off | Disconnected | 0 watt |
DC | Suspend to RAM | Disconnected | 0-1 watt | The system is able to resume successfully. Power meter reports 0 watt, so consumption is likely closer to 0 watt but unequal to 0.
DC | Suspend to Idle | Disconnected | 2 watt | 
DC | On | Disconnected | 3 watt | Idle, display at minimal brightness
DC | On | Disconnected | 3 watt | Idle, display at 50% brightness
DC | On | Disconnected | 4 watt | Idle, display at 75% brightness
DC | On | Disconnected | 5 watt | Idle, display at maximum brightness
DC | On | Disconnected | 7-9 watt | Playing a video (YouTube in Firefox, 1080p@30hz, H.264 and VP9)
DC | On | Disconnected | 10-12 watt | Playing a video (dito) and running `stress --cpu 6`

Conclusions:
- Power over the DC connector seems to be capped at 15 watt (5V 3A), probably by the notebook. Testing with a more robust power supply might confirm this.
- Power over USB or USB PD seems to be hard capped by the notebook around 12 watt, probably at 5V 2.5A. In some other tests with USB-C docks, measurements indicated that the maximum power can be much lower depending on the dock.
- The notebook can use up to 12 watt to run the hardware, excluding charging the battery. This means that with a good USB-based power supply, the battery will barely charge if the device is doing something intensively. With a lesser USB-based power supply (or a good power supply combined with a crappy USB-C dock), the battery will start to discharge because not enough power will be available/negotiated.

Advice:
- First of all, [never connect two power supplies at the same time](https://wiki.pine64.org/index.php/Pinebook_Pro#Power_Supply)!
- Use the included power supply, or a similar one with the same DC connector and with the same (or better) specifications. This way, the battery will always be able to charge, even when the system is under heavy load.
- Avoid using a USB (PD) power supply, especially when using a USB-C dock. One exception is when you use a lot of connected USB accessoires, which the Pinebook Pro might not be able to power all by itself. In thase case, ensure you use a good USB PD power supply and a good USB-C dock. The [official dock](https://pine64.com/product/pinebook-pro-usb-c-docking-deck/?v=0446c16e2e66) might give good results, but it has not been tested.
- Consider reducing the screen brightness to 50% to speed up charging a bit when the system is busy and the battery is not full.

Some useful values that can be read by the Pinebook Pro itself for further testing:

Path | Function
--- | ---
`/sys/class/power_supply/cw2015-battery/capacity` | Battery charge in percentage
`/sys/class/power_supply/cw2015-battery/charge_counter` | How often the battery controller switched to charging mode due to a DC power supply being connected
`/sys/class/power_supply/cw2015-battery/current_now` | Discharge current
`/sys/class/power_supply/cw2015-battery/status` | Status (charging, discharging, ...)
`/sys/class/power_supply/cw2015-battery/time_to_empty_now` | Estimated full discharge time in minutes
`/sys/class/power_supply/cw2015-battery/voltage_now` | Charge and discharge voltage
`/sys/class/power_supply/dc-charger/online` | Power supply connection status (DC, USB and USB PD)
`/sys/class/power_supply/tcpm-source-psy-4-0022/current_max` | Negotiated maximum power (USB PD only)
`/sys/class/power_supply/tcpm-source-psy-4-0022/current_now` | Used power, estimation (USB PD only)


## Why does the system consume so much energy when sleeping?

For the Pinebook Pro, Arch Linux ARM is configured by default to use [Suspend to Idle](https://www.kernel.org/doc/html/v5.12/admin-guide/pm/sleep-states.html#suspend-to-idle), which is not energy efficient. After suspending and resuming (e.g. by running `systemctl suspend`), `dmesg` will report **PM: suspend entry (s2idle)**. This indicates that Suspend to Idle was used.

[Suspend to RAM](https://www.kernel.org/doc/html/v5.12/admin-guide/pm/sleep-states.html#suspend-to-ram) support on the Pinebook Pro was unavailable for a long time. A [fix](https://review.trustedfirmware.org/c/TF-A/trusted-firmware-a/+/9616) for proper Suspend to Sleep support was added to [Trusted Firmware A](https://developer.trustedfirmware.org/project/profile/1/), which is part of the Pinebook Pro's bootloader. This fix has not yet been implemented in the Das U-Boot bootloader included in Manjaro ARM.

[Tow-Boot](https://github.com/Tow-Boot/Tow-Boot) is a fork of U-Boot, and contains the fixed Trusted Firmware A. It is included in the Pinebook Pro root filesystem offered in the [Releases section](releases), and the [installation instructions](INSTALL.md) have it installed by default.

Summarized, it is possible to enable Suspend to RAM under the following conditions:

- Tow-Boot is installed, which it is by default.
- No NVMe SSD is installed. Suspend to RAM with an NVMe SSD [is currently not supported](https://forum.pine64.org/showthread.php?tid=11380).

To enable Suspend to RAM, run as root:

```
mv /etc/systemd/sleep.conf.d/suspend2idle.conf{,.disabled}
```

After suspending and resuming, `dmesg` will report **PM: suspend (deep)**. This indicates that Suspend to RAM was used.

Note that suspend to RAM is still experimental. The system might refuse to go to sleep. See the reason with `dmesg`, and consider reporting it upstream. If the system refuses to go to sleep, simply try it again. It can also be that the system refuses to wake up. In that case, press and hold the power button for 10 seconds, release, and press it again briefly to restart the system.

  
## Why is wireless networking performing poorly or not at all, especially on 5 GHz bands?

The Pinebook Pro misses information about which wireless frequencies are allowed to be used in your region. To specify the region, install package `crda` and uncomment the line that relates to the correct country in `/etc/conf.d/wireless-regdom`. Reboot after the modification to apply the change.


## Why is USB-C video output not working?

If you use a [compatible](https://wiki.pine64.org/wiki/Pinebook_Pro_Hardware_Accessory_Compatibility#USB_C_alternate_mode_DP) USB-C dock, connect the monitor first to the dock, then connect the dock to the Pinebook Pro. Also, disconnect and reconnect the dock after boot once if video output is not working.

Note that [HDMI audio is not working at this time](https://forum.manjaro.org/t/no-hdmi-audio-on-pinebook-pro/50203/2).


## Why is the maximum audio volume so low?

The root filesystem ships with the semi-official recommended volume settings.  You can boost the volume by opening `alsamixer` and increasing 'headphone' volume (the first volume bar). Note that there are four settings, which control the amplification before the analog audio gets send to either speakers or headphones. dB gain -48 is the default. -24 seems to be equal to -48. -12 boosts the volume, and makes the audio clip on the internal speakers on maximum volume. With -0, clipping starts above 50% volume on the internal speakers.
  
Remember that bad sound kills good music, and tinnitus sounds awful. Be careful when messing around with volumes, especially with headphones.


## Why is Bluetooth A2DP audio not working?

This is due to [a bug in bluez](https://github.com/bluez/bluez/issues/157), affecting not just the Pinebook Pro. The current workaround is to install [bluez-git](https://aur.archlinux.org/packages/bluez-git/) from AUR, stop `bluetooth.service`, remove `/var/lib/bluetooth`,  and start `bluetooth.service`. After this, pair your Bluetooth audio device again.

The workaround was tested with [PipeWire](https://wiki.archlinux.org/title/PipeWire) and [pipewire-pulse](https://archlinux.org/packages/?name=pipewire-pulse).


## Why do videos not play smoothly?

Currently, the Pinebook Pro [lacks full video hardware acceleration](https://forum.manjaro.org/t/pinebook-pro-hardware-acceleration/30136). The [driver](https://wiki.pine64.org/wiki/Pinebook_Pro#Hardware_video_decoding) is there, [but VA-API support on modern kernels is currently missing](https://github.com/bootlin/libva-v4l2-request/issues/35#issuecomment-843147661).

The CPU of the Pinebook Pro is powerful enough to decode H.264 in 1080p at standard frame rates (24-30 FPS) in software. For browsers install Enhancer for YouTube ([Firefox](https://addons.mozilla.org/en-US/firefox/addon/enhancer-for-youtube/), [Chromium](https://chrome.google.com/webstore/detail/enhancer-for-youtube/ponfpcnoihfmfllpaingbgckeeldkhle)), and configure it to force H.264 playback and to only use standard frame rate formats. This will allow smooth YouTube playback.


## How can I install application XYZ?

This is not a Pinebook Pro specific question. To learn more about Arch Linux (on which Arch Linux ARM is based), check the [ArchWiki](https://wiki.archlinux.org/). If you cannot find your package in the [Arch Linux ARM package list](https://archlinuxarm.org/packages), and you cannot get it (installed) from the [Arch User Repository](https://aur.archlinux.org/), you might need to find some other way to get your application working.

Some tips from personal experience:

- Signal Desktop can be installed from the [PrivacyShark repository](https://privacyshark.zero-credibility.net/). Audio and video calls work. The repository also has other interesting software. Its [package descriptions can be found here](https://gitlab.com/ohfp/pinebookpro-things).
- Skype can be used through [Skype for Web](https://web.skype.com/) in Chromium. Audio and video calls work. Make sure to [enable video acceleration](#why-is-youtube-so-slow) for best results.
