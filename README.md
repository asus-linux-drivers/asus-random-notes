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

# Set Battery charging threshold 

Credit: https://www.linuxuprising.com/2021/02/how-to-limit-battery-charging-set.html

```
sudo touch /etc/systemd/system/battery-charge-threshold.service
sudo gedit /etc/systemd/system/battery-charge-threshold.service
```


- Get BATTERY_NAME from `ls /sys/class/power_supply`
- Set CHARGE_STOP_THRESHOLD as the battery percentage where you want charging to stop, eg for 80, give 81

```
[Unit]
Description=Set the battery charge threshold
After=multi-user.target
StartLimitBurst=0

[Service]
Type=oneshot
Restart=on-failure
ExecStart=/bin/bash -c 'echo CHARGE_STOP_THRESHOLD > /sys/class/power_supply/BATTERY_NAME/charge_control_end_threshold'

[Install]
WantedBy=multi-user.target
```
    

Then enable it

```
sudo systemctl enable battery-charge-threshold.service
sudo systemctl start battery-charge-threshold.service
```


Edit `/etc/systemd/system/battery-charge-threshold.service` to change threshold whenever needed 

```
sudo systemctl daemon-reload
sudo systemctl restart battery-charge-threshold.service
```

