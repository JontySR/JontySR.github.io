---
title: Using SSH and Internet Sharing with the Pi Zero Over USB (USB Gadget)
layout: post
date: 2021-01-05
categories: pi
tags: raspberry pi usb gadget ethernet zero internet sharing ssh
---

Please note that I'm using Linux on my PC.
If you use Windows or macOS, the changes made to the PC in the later stages will be different, but the gist is the same.
There are multiple points in this guide where you can stop early if you don't want all the features.

## Setting up your Pi as a USB gadget

### Preparing the micro SD card

Mount the micro SD card on your PC, open the volume named *boot* and edit the file `config.txt`, adding a new line at the bottom of the file containing the text `dtoverlay=dwc2`.

In the same volume, edit the file `cmdline.txt`, adding `modules-load=dwc2,g_ether` after `rootwait` on the same line in that file.
Make sure there's a single space between our new text and `rootwait` and a space after our new text if there's any text following it.
These changes allow the Pi to be a USB gadget, but it wouldn't be much use if we couldn't SSH into it.

Still in *boot*, create a new empty file called `ssh`.

### Plugging the Pi in

Plug a micro USB cable into the port marked "USB" (furthest from the end of the board) and not "PWR IN", with the other end of the cable plugged into your PC.
We only need one cable because this will power the board sufficiently at the same time as allowing data transfer over the cable.
On your PC, run `lsusb` to see if our device turns up as a USB gadget - you'll likely see the acronym RNDIS.

If after a minute or so, your Pi is not detected as a USB gadget, note that some micro USB cables will not allow data transfer, so swap it out with another if you have one to hand, or buy one online that advertises this.
Have a look in your network connections to see if this device turns up as an ethernet connection too; this will be our way in.

At this point, you could just change your new ethernet network entry in it's IPv4 settings to "Link-Local Only", disconnect and reconnect, and SSH into the Pi with the command `ssh pi@raspberrypi.local`.
This would be fine if you didn't need Internet access.

## Sharing Internet with your Pi

This bit's slightly annoying.
We need Internet on our Pi to get `dnsmasq`, which will allow us to hand out IPs from the Pi, which is the next step in improving our USB gadget's usability.
I've written down below about why I'm not using "Shared with other computers" and leaving it there.

Unplug the Pi again and open up the micro SD on your PC.
This time, go into the volume named *root* on the card and navigate through to `etc/network/interfaces`.
Add the following lines to the end of the file:

```
auto usb0
iface usb0 inet static
  address 10.42.0.2
  netmask 255.255.255.0
```

The changes we just made set a static IP on the Pi of 10.42.0.2.
This is done because the IP of your PC when using "Shared with other computers" (for our first connection) on the USB ethernet network will be 10.42.0.1, and we want the Pi to be predictably within that subnet but not clashing with the IP of the PC.

You can leave it here if you don't mind not being able to address the Pi with its hostname when using SSH, and you won't have multiple Pi USB gadgets plugged in at once (the next gadget will be on the subnet 10.42.1.0).
It's quite a rigid solution, but it'll work if you just need it for one Pi.

### Making it a bit less temperamental

#### dnsmasq

Let's make this solution a bit more usable and future-proof by using our new Internet connection on the Pi to get dnsmasq.

Stay SSH'd into the Pi from the last section and run `sudo apt install dnsmasq` (I'm assuming you've already done `sudo apt update && sudo apt upgrade` for a fresh install).

Once dnsmasq is installed, open `/etc/dnsmasq.conf` and add the following lines to the end of the file:

```
interface=usb0
dhcp-range=usb0,10.42.0.2,10.42.0.5,255.255.255.0,1h
dhcp-option=3
```

And change `/etc/network/interfaces` to use a static IP of 10.42.0.1.
Make sure dnsmasq is enabled with `sudo systemctl enable dnsmasq`.
dhcpcd won't run because a static IP is set, but if you want to disable it altogether, run `sudo systemctl disable dhcpcd`.

What we've done there is told the Pi to provision devices which join its network (your PC) with an IP in the range of 10.42.0.2 and 10.42.0.5, while the Pi will have a static IP of 10.42.0.1.
This ensures that the Pi and your PC will always be in the same subnet when connected, the connection can (and must) be set back to "Automatic (DHCP)" and you should be able to SSH into the Pi with its hostname.

Sadly, we need to share Internet with it using a different method.

#### ufw

We can use ufw on the PC to share the Internet with the Pi.
Install ufw on your distribution of Linux.

Open up `/etc/ufw/sysctl.conf` on your PC and uncomment the following line:

```
net.ipv4.ip_forward=1
```

And find the line in `/etc/default/ufw` containing the text `DEFAULT_FORWARD_POLICY=` and change whatever comes after the `=` to `ACCEPT`.
It should read:

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

Note down the name of your WiFi interface on your PC, found by running `ip link` (it'll start with `wlp`).
In `/etc/ufw/before.rules` (still on your PC, not the Pi), add the following lines before the `*filter` rules:

```
*nat
:POSTROUTING ACCEPT [0:0]

-A POSTROUTING -o <WIRELESS INTERFACE> -j MASQUERADE

COMMIT
```

Finally, make sure ufw is restarted using `sudo systemctl restart ufw && sudo systemctl force-reload ufw` and that it's enabled using `sudo ufw enable`.

Now reboot the Pi and it should be SSH-able by its hostname, and have access to the internet.
If you're having issues accessing the internet, check that it's not a problem with DNS (read below).

All done!
You've finally got a setup worthy of exhausting me out!

## Extra

### Why not "Shared with other computers"?

One thing many other guides suggest is to set the new network entry's settings on your PC to "Shared with other computers" to share Internet.
This only works maybe 1 in 10 times for me, as the Pi keeps setting its address to 169.254.x.x and not 10.42.0.x.
It also prevents using the hostname to SSH into the Pi.

### Running into DNS issues

Can `ping 8.8.8.8` but not `ping google.com`?
You've got DNS issues.
Add the following to the end of the indented section after `iface usb0 inet static`:

```
dns-nameservers 8.8.8.8 8.8.4.4
dns-search domain-name
```

Now reboot or risk running `sudo service networking restart` on the Pi and hope you stay connected.

### Connecting multiple Pi's to the same PC

Change the subnet to be 10.42.1.x in the above configs for a second Pi so its DHCP addresses don't clash with the first for example.

### Sources

- [USB Gadget](https://unix.stackexchange.com/questions/575178/sharing-wifi-internet-through-ethernet-interface)
- [DNS Fix](https://pimylifeup.com/raspberry-pi-dns-settings/)
- [dnsmasq Config](https://pimylifeup.com/raspberry-pi-dns-settings/)
- [WiFi Sharing with ufw](https://unix.stackexchange.com/questions/575178/sharing-wifi-internet-through-ethernet-interface)
