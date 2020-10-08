# Cage / Wayland linux kiosk setup tutorial

## About

In this tutorial you will learn how to setup an effecient, minimalist linux kiosk system utilizing Cage, a state of the art Wayland compositor designed specifically for kiosk use. This tutorial wil not be exaustive but will get you up and running with the minimum required software.

## Step 1: Install Arch linux on you target machine following their guide.

https://wiki.archlinux.org/index.php/installation_guide

or for arm systems like raspberry pi 4

https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-4

## Step 2: Update packages

```
pacman -Syyuu
```

## Step 3: Install dependencies

```
pacman -S at-spi2-atk atk cage chromium gdk-pixbuf2 gtk3 libxcomposite libxcursor libxi libxtst networkmanager sudo xorg-server-xwayland
```

## Step 4: Enable auto login

override the tty1 getty service via

```
systemctl edit getty@tty1
```

Add the following lines to the file. Save and exit.

```
[Service]
Type=simple
ExecStart=
ExecStart=-/usr/bin/agetty --autologin <YOUR_USER> --noclear %I $TERM
```

## Step 5: Auto start your kiosk application

You can start cage (and therefore your kiosk application) automatically without a login manager, for example, by adding this to the end of ~/.bash_profile (~/.zlogin or ~/.zprofile for Zsh):

```
if [ "$(tty)" = "/dev/tty1" ]; then
    export WLR_LIBINPUT_NO_DEVICES=1 # required for systems that doesn't have a keyboard or mouse.
	exec cage -d  -- <YOUR_APPLICATION>
fi
```

or for a simple web kiosk

```
if [ "$(tty)" = "/dev/tty1" ]; then
	exec cage -d -- chromium <YOUR_CHROMIUM_FLAGS>
fi
```

## Further reading

https://github.com/Hjdskes/cage/wiki/Configuration

https://github.com/swaywm/sway/wiki

https://kapeli.com/cheat_sheets/Chromium_Command_Line_Switches.docset/Contents/Resources/Documents/index
