![Badge](https://hitscounter.dev/api/hit?url=https%3A%2F%2Fgithub.com%2Fasus-linux-drivers%2Fasus-random-notes&label=Visitors&icon=suit-heart-fill&color=%23e35d6a)
[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fasus-linux-drivers%2Fasus-random-notes&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

# BIOS upgrade (with EZ Flash)

```
# current version
$ sudo dmidecode -s bios-version
UP5401EA.300
```

- Download the latest BIOS for ASUS EZ Flash Utility (303) from laptop specific [Asus website](https://www.asus.com/laptops/for-home/zenbook/zenbook-14-flip-oled-up5401-11th-gen-intel/helpdesk_bios/?model2Name=Zenbook-14-Flip-OLED-UP5401-11th-Gen-Intel)
- Extract
- Format USB flash disk to FAT32 (`$ sudo apt-get install gparted` or `$ sudo apt-get install gnome-disk-utility`)
- Copy file `UP5401EAAS.303` from extracted folder to root of USB flash disk (does not have to be empty)
- Restart laptop and hold BIOS entry key F2
- Advanced mode -> Advanced -> Asus EZ Flash 3 Utility -> select USB Flash disk (orientate via capacity) -> Enter

# Firmware update (with fwupd)

> This project aims to make updating firmware on Linux automatic, safe, and reliable.

*(https://github.com/fwupd/fwupd)*

```
$ sudo apt-get install -y fwupd
$
$ fwupdmgr refresh
Firmware metadata last refresh: 1 minute ago. Use --force to refresh again.
$ fwupdmgr refresh --force
Updating lvfs
Downloading…             [***************************************]
Successfully downloaded new metadata: 0 local devices supported
$ fwupdmgr update
$
$ fwupdmgr get-devices # list devices with currently installed firmware
$ fwupdmgr --help
```

# Export of DSDT table as *.dsl

```
$ sudo apt-get install iasl
$ sudo cp /sys/firmware/acpi/tables/DSDT DSDT
$ sudo iasl -d DSDT
$ cat DSDT.dsl
```

# Locked Fn key whenever laptop starts

- Add these two lines to the file `sudo gedit /etc/modprobe.d/alsa-base.conf`

```
# Toggle fnlock_default at boot (Y/N)
options asus_wmi fnlock_default=N
```

- Reload & reboot

```
$ sudo update-initramfs -u -k all; reboot
```

# Usage of Intel graphics

```
$ sudo apt-get install -y intel-gpu-tools
$ sudo intel_gpu_top
```

# 32 or 64-bit OS
```
$ getconf LONG_BIT
```

# HP printer

- Download and install latest `*.run` file from [developers.hp.com](https://developers.hp.com/hp-linux-imaging-and-printing/gethplip)
```
$ chmod +x *.run
$ sudo ./*.run
# reboot, install script will ask at the end
```

- Create hotspot on mobile (connect printer to hotspot, connect laptop to hotspot)
- Setup printer
```
$ sudo hp-setup
```

*Using phone app HP Smart (with plug-in module for HP Printer) has to be started data connection at first but for printing disabled, for scanning has not to be disconnected connection to internet*

# Stuck on login screen

- Display terminal `CTRL + ALT + F3`
- Back to login screen `CTRL + ALT + F7`

# Fingerprint

> You will be able to use your fingerprint scanner to authenticate to elevated privileges, ie. sudo. It will also allow you to login and unlock your system using the fingerprint reader.

*(https://askubuntu.com/a/1243194)*

- Check whether is a fingerprint reader available `$ lsusb`
- Install fingerprint driver
```
$ sudo apt install fprintd libpam-fprintd
```
- Enroll 5 times your fingerprint (`right-index-finger` is default)
```
$ sudo fprintd-enroll -f [finger_name]
```
- Verify scanned fingerprint (`right-index-finger` is default)
```
$ sudo fprintd-verify -f [finger_name]
```
- Enable fingerprint authentication
```
$ sudo pam-auth-update --enable fprintd
```

# Wifi does not work

- Connect laptop via ethernet or created phone hotspot tethered via USB to internet (Android: Settings -> Wireless connection and networks -> Tethering and portable hotspot -> enable checkbox Tethering via USB)
- Install wifi driver
```
$ sudo apt install rtl8821ce-dkms
```
- Reboot

# Disable bluetooth on startup

## Using TLP

- Find and set `DEVICES_TO_DISABLE_ON_STARTUP="bluetooth"` in config `sudo gedit /etc/tlp.conf`
- Apply

```
# do a config reload and restart
$ sudo tlp start
```

## Script

```
#!/bin/sh

sudo systemctl stop bluetooth.service
bluetooth off
```

# Set Battery charging thresholds

> The purpose of battery care on laptops is to reduce the loss of capacity due to wear from ongoing battery operation i.e. to prolongate the battery lifespan. This can be achieved by:
>
> - Limiting the maximum charge level to below 100%: stop charge threshold
>
> - After a short discharge, prevent the charging process from continuing as soon as the charger is connected: start charge threshold
>
> - Recalibration: helps keeping charge level (SOC) readings and battery runtime estimates accurate by setting new “full charge” and “full discharge” anchors in the battery controller

*(https://linrunner.de/tlp/settings/battery.html#battery-care)*

## Using TLP (how to customize End & Start charge threshold when supported)

- Check TLP version, for Asus you need TLP 1.4 or newer [TLP changelog](https://github.com/linrunner/TLP/blob/main/changelog#L70)
- Get battery number from `ls /sys/class/power_supply`
- Check whether your hardware supports changing both values via existing files `cat /sys/class/power_supply/BAT0|1/charge_control_end_threshold|charge_control_start_threshold`
- If your hardware supports only a stop threshold, set in next step the start value to 0
- Find and set `START_CHARGE_THRESH_BAT0|1`, `STOP_CHARGE_THRESH_BAT0|1` in config `sudo gedit /etc/tlp.conf`
- Apply newly set limits

```
# do a config reload and restart
$ sudo tlp start
```

## Systemd service (how to customize End charge threshold)

- Get `BATTERY` from `$ ls /sys/class/power_supply`
- Check whether your hardware supports changing end charge threshold via existing file `$ cat /sys/class/power_supply/BAT0|1/charge_control_end_threshold`
- Set `CHARGE_END_THRESHOLD` as the battery percentage where you want charging to stop (eg. for 80, give 81)
- Create service `$ sudo gedit /etc/systemd/system/battery-charge-threshold.service`

```
[Unit]
Description=Set the battery charge threshold
After=multi-user.target
StartLimitBurst=0

[Service]
Type=oneshot
Restart=on-failure
ExecStart=/usr/bin/env bash -c 'echo $CHARGE_END_THRESHOLD > /sys/class/power_supply/$BATTERY/charge_control_end_threshold'
Environment="BATTERY=BAT0"
Environment="CHARGE_END_THRESHOLD=90"

[Install]
WantedBy=multi-user.target
```

- Then enable and start service

```
# --now is equivalent for sudo systemctl start
$ sudo systemctl enable --now battery-charge-threshold.service
```

- Edit `/etc/systemd/system/battery-charge-threshold.service` to change threshold whenever needed, for applying immediately and not after restart is necessary reload service files and restart service

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart battery-charge-threshold.service
```

# Allow charging phone via USB

## When is TLP running

> Symptom: some USB devices do not work reliable when TLP activates USB autosuspend mode.
>
> All input devices (driver usbhid), libsane-supported scanners and (as of version 1.4) audio devices get excluded by default. It’s therefore unnecessary to put them on the USB_DENYLIST. To circumvent the default for certain devices enter the IDs into USB_ALLOWLIST.

*(https://linrunner.de/tlp/faq/usb.html#usb-devices)*

- Connect device and get ID **2717:ff40**
```
$ lsusb
$ tlp-stat -u # or with this
```

- Add obtained ID in line `USB_ALLOWLIST="2717:ff40"` to config `$ sudo gedit /etc/tlp.conf`
 
*(This parameter was renamed with version 1.4. Until 1.3 it was called USB_WHITELIST. 1.4 and newer also recognize the old name.)*
 
- Apply changes
 
```
# do a config reload and restart
$ sudo tlp start
```
