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

## Using TLP (how to customize Start & End charge threshold)

- Get battery number from `ls /sys/class/power_supply`
- Find and set `START_CHARGE_THRESH_BAT0|1`, `STOP_CHARGE_THRESH_BAT0|1` in config `sudo gedit /etc/tlp.conf`
- Apply newly set limits

```
sudo tlp start
```

## Systemd service (how to customize End charge threshold)

- Get `BATTERY` from `ls /sys/class/power_supply`
- Set `CHARGE_END_THRESHOLD` as the battery percentage where you want charging to stop (eg. for 80, give 81)
- Create service `sudo gedit /etc/systemd/system/battery-charge-threshold.service`

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

