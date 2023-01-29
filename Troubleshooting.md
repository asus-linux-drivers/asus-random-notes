# 1. Asus laptop screen doesn't wake from suspend

> The issue is `mostly` fixed in latest kernels and thus latest distributions

For instance machines (this has nothing to do whether you have an Nvidia GPU or not, its broken in AMD iGPUs too) running Ubuntu 20.04 and based distributions are affected. The only way to get your laptop working back is holding the power button and thus **rebooting it forcefully**, while **losing unsaved work**. Even entering to TTYs aren't possible.

There is no solution, the best you can do is to prevent loss of work is **disabling suspend feature altogether, even when you close the lid**

```
For Gnome Users, 

1. Set "blank screen" to "never" in settings>power

2. Set "automatic suspend" to "off" in settings>power

3. Set "power button action" to "Nothing" from "Suspend"

Do the equivalent for other DEs too, if you aren't using Gnome
```

Follow the below to prevent system from suspending when laptop lid is closed

```
Open logind.conf file in etc/systemd using gedit

sudo gedit /etc/systemd/logind.conf

Then uncomment (remove the #) the line containing "HandleLidSwitch" and set its value to "ignore"

Reboot
```
