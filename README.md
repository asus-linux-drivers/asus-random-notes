# Fingerprint

> You will be able to use your fingerprint scanner to authenticate to elevated privileges, ie. sudo. It will also allow you to login and unlock your system using the fingerprint reader.

*(https://askubuntu.com/a/1243194)*

- Check whether is a fingerprint reader available `$ lsusb`
- Install fingerprint driver
```
$ sudo apt install fprintd libpam-fprintd
```
- Enroll 5 times your fingerprint
```
$ fprintd-enroll
```
- Enable fingerprint authentication (using key `Space`)
```
$ sudo pam-auth-update
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

# BIOS upgrade (with EZ Flash)

```
# current version
$ sudo dmidecode -s bios-version
UP5401EA.300
```

- Download the latest BIOS (303) from laptop specific [Asus website](https://www.asus.com/laptops/for-home/zenbook/zenbook-14-flip-oled-up5401-11th-gen-intel/helpdesk_bios/?model2Name=Zenbook-14-Flip-OLED-UP5401-11th-Gen-Intel)
- Extract
- Copy from extracted folder file `UP5401EAAS.303` to USB flash disk
- Restart laptop and hold BIOS entry key F2
- Advanced mode -> Advanced -> Asus EZ Flash 3 Utility -> select USB Flash disk (orientate via capacity) -> Enter

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
ExecStart=/bin/bash -c 'echo $CHARGE_END_THRESHOLD > /sys/class/power_supply/$BATTERY/charge_control_end_threshold'
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
```
- Add obtained ID in line `USB_DENYLIST="2717:ff40"` to config `$ sudo gedit /etc/tlp.conf`
- Apply changes

```
# do a config reload and restart
$ sudo tlp start
```
