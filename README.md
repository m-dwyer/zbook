# HP ZBook Studio x360 G5 - Linux

## Current Setup

* Arch Linux 5.2.13
* GNOME + GDM

## Known Issues

### Nouveau Causing Crashes

After installing the `nvidia` package along with `xorg-server` and `gnome`, I noticed the `nouveau` kernel module seems to load and cause freezing, sometimes generating a kernel panic in another tty.

#### Fix

Blacklist the `nouveau` module, and add the `install` option so any attempts to load will just return `true` instead.

In `/etc/modprobe.d/nouveau.conf` or similar:
```
blacklist nouveau
install nouveau /bin/true
```

### Nvidia module not loading

Nvidia module was failing to load (I don't have the exact error message at hand sorry).

#### Fix

Enabling kernel modesetting for nvidia driver in `/etc/modprobe.d/nvidia-kms.conf` or similar:
```
options nvidia-drm modeset=1
```

### GPU switching

I decided on [optimus-manager](https://github.com/Askannz/optimus-manager) to allow switching between onboard (intel) and dedicated GPU (nvidia). However, something outside of udev and my initramfs was causing the nvidia kernel module to load when blacklisted (despite a clean xorg config).

I also decided to keep GDM, as changing away from GDM to LightDM breaks power management (screen dim/off and locking).  I didn't want to rely on a patched GDM (available in the AUR, based on the patch Ubuntu uses) -- so I've opted to reboot when changing GPU.

When in intel mode, optimus-manager appears to rely on a static modprobe blacklist file, and a systemd service then will load the nvidia modules manually.  However, something (?) on my system (I checked my initramfs) causes the `nvidia` module to load

The workaround is to blacklist and add install options for modprobe.  I'm doing this manually for now, but working on something similar to [fedora-prime](https://github.com/m-dwyer/fedora-prime) for this, as I don't believe optimus-manager will work in its current form.

In `/etc/modprobe.d/nvidia.conf` or similar:
```
blacklist nvidia_drm
blacklist nvidia_uvm
blacklist nvidia_modeset
blacklist nvidia
install nvidia_drm /bin/true
install nvidia_uvm /bin/true
install nvidia_modeset /bin/true
install nvidia /bin/true
```

The blacklist options will not stop modules loading after boot/udev hardware detection.  The `install` option ensures that if something *does* try to load the module, `true` will simply be returned instead.

### Brightness Up/Down Hotkeys

Originally on Ubuntu 19.04 and *some* 5.x.x kernel versions (I can't remember exact versions) on Fedora, brightness hotkeys were not working.

When pressed, brightness up/down hotkeys were registering as microphone mute/unmute.  I checked `showkey`, `showkey --scancodes` and `evtest`, all which indicated the brightness keys were registering the mic mute scancode.

After messing around a little looking through kernel driver source, attempting to decompile and check ACPI tables, and write my own WMI kernel module to listen for WMI events -- I wasn't successful in finding the bug.

#### Fix

I decided to move back to Arch for a number of reasons, at which point one of the 5.2.x kernels had resolved this bug (I'm still curious to know where it was, as I'm not very familiar with kernel dev).

### Touchpad not working on resume

When suspending and resuming, the touchpad does not work.  Workaround:

`#rmmod i2c_hid && modprobe i2c_hid`

Searching around, there are some systemd unit files to automate this.