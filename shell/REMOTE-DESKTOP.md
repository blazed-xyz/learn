# Remote Desktop (RDP)

First, make sure to enable remote desktop.

Next, apply the following patches to avoid conflict with the console machine:

1. Create console user
```sh
sudo useradd -m -s /bin/bash console
sudo passwd console
sudo usermod -aG wheel console
```

```sh
sudo nano /etc/gdm/custom.conf
```

Add the following to the "daemon" section:
```
[daemon]
AutomaticLoginEnable=True
AutomaticLogin=console
WaylandEnable=false
```

**Make GNOME the default boot target**

```sh
sudo systemctl set-default graphical.target
sudo systemctl enable gdm
```

Verify:

```sh
systemctl get-default
```

Reboot:

```sh
sudo reboot
```