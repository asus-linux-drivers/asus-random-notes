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
$ tlp-stat -u # or
```

- Add obtained ID in line `USB_ALLOWLIST="2717:ff40"` to config `$ sudo gedit /etc/tlp.conf`
 
*(This parameter was renamed with version 1.4. Until 1.3 it was called USB_WHITELIST. 1.4 and newer also recognize the old name.)*
 
- Apply changes
 
```
# do a config reload and restart
$ sudo tlp start
```
