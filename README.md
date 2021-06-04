# Cage / Wayland linux kiosk setup tutorial

## About

In this tutorial you will learn how to setup an effecient, minimalist linux kiosk system utilizing Cage, a state of the art Wayland compositor designed specifically for kiosk use. This tutorial wil not be exhaustive but will get you up and running with the minimum required software.

## Step 1: Install Arch linux on your target machine following their guide.

https://wiki.archlinux.org/index.php/installation_guide

or for arm systems like raspberry pi 4

https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-4

## Step 2: Update and install dependencies

```
pacman -Syyuu cage chromium networkmanager sudo xorg-xwayland
```

## Step 3: Starting Cage on boot with systemd

On systemd-based Linux distributions, systemd can be used to automatically start a Cage session when the system boots. This achieves the typical embedded application setup where starting the device automatically lands you in the application. Note that for this to work, wlroots must be built with systemd logind support.

In order to integrate Cage in the boot process like this, we need to configure a number of things.

First, the systemd unit file. Place the following template unit file in
```
/etc/systemd/system/cage@.service
```
```
# This is a system unit for launching Cage with auto-login as the
# user configured here. For this to work, wlroots must be built
# with systemd logind support.

[Unit]
Description=Cage Wayland compositor on %I
# Make sure we are started after logins are permitted. If Plymouth is
# used, we want to start when it is on its way out.
After=systemd-user-sessions.service plymouth-quit-wait.service
# Since we are part of the graphical session, make sure we are started
# before it is complete.
Before=graphical.target
# On systems without virtual consoles, do not start.
ConditionPathExists=/dev/tty0
# D-Bus is necessary for contacting logind, which is required.
Wants=dbus.socket systemd-logind.service
After=dbus.socket systemd-logind.service
# Replace any (a)getty that may have spawned, since we log in
# automatically.
Conflicts=getty@%i.service
After=getty@%i.service

[Service]
Type=simple
# Required if device doesn't have a keyboard or mouse
Environment=WLR_LIBINPUT_NO_DEVICES=1
# Disable cursor
Environment=XCURSOR_PATH=0
ExecStart=/usr/bin/cage /usr/bin/chromium
Restart=always
User=cage
# Log this user with utmp, letting it show up with commands 'w' and
# 'who'. This is needed since we replace (a)getty.
UtmpIdentifier=%I
UtmpMode=user
# A virtual terminal is needed.
TTYPath=/dev/%I
TTYReset=yes
TTYVHangup=yes
TTYVTDisallocate=yes
# Fail to start if not controlling the virtual terminal.
StandardInput=tty-fail

# Set up a full (custom) user session for the user, required by Cage.
PAMName=cage

[Install]
WantedBy=graphical.target
Alias=display-manager.service
DefaultInstance=tty7
```

In order for the unit file to work properly, you'll need to ensure that the specified user exists. Any normal Linux user suffices. Don't forget to swap out ```/usr/bin/chromium``` with your application of choice!

Next, we need to create the custom PAM stack that is invoked on login. This stack needs to authenticate the user and register the user session with the systemd login manager. The below file is a minimal working configuration that can be dropped in ```/etc/pam.d/cage```:
```
auth           required        pam_unix.so nullok
account        required        pam_unix.so
session        required        pam_unix.so
session        required        pam_systemd.so
```

Third, enable an instantiated service with ```systemctl enable cage@tty1.service``` (or any other TTY).

Finally, change systemd's default target to the graphical target with ```systemctl set-default graphical.target```.


Reference: https://github.com/Hjdskes/cage/wiki/Starting-Cage-on-boot-with-systemd

## Further reading

https://github.com/swaywm/sway/wiki

https://github.com/Hjdskes/cage/wiki/Configuration

https://kapeli.com/cheat_sheets/Chromium_Command_Line_Switches.docset/Contents/Resources/Documents/index
