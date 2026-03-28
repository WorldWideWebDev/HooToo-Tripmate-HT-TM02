# HooToo-Tripmate-HT-TM02
# HooToo TripMate HT‑TM02 — Full Recovery & LEDE/OpenWrt Resurrection Guide 

Before we start, 
How do i open a Hootoo? How do i connect the hootoo? If you need guidance, scroll down to resources,  

The video in the resources section will help you open, connect and get wired up.

HooToo-HT-TM02-Recovery
# HooToo TripMate HT‑TM02 — Full Recovery & LEDE/OpenWrt Resurrection Guide

This repository documents the **complete, real‑world recovery process** for the HooToo TripMate HT‑TM02 — including serial access, initramfs booting, raw MTD flashing, and final configuration.  
Most guides online are outdated, incomplete, or assume different hardware revisions.  
This one is based on an **actual, successful recovery** performed in 2026.

If your HT‑TM02 is:
- stuck with no Wi‑Fi  
- missing the web UI  
- not giving out DHCP  
- soft‑bricked  
- only reachable via serial  
- or booting but unusable
- both led lights constantly on, with other signs of life

…this guide will get it back to life.

---

## 🧠 Hardware Overview

| Component | Spec |
|----------|------|
| SoC | Ralink RT5350F @ 360 MHz |
| RAM | 32 MB |
| Flash | 8 MB SPI NOR |
| Wi‑Fi | 2.4 GHz 802.11n (1T1R, 150 Mbps PHY) |
| Ethernet | 1× 100 Mbps |
| USB | None |

Real‑world Wi‑Fi throughput: **20–30 Mbps** (CPU‑bound)

---

## 🔌 Serial Console Access

### Pinout (from left to right, with Ethernet port facing away):
- **3.3V** (do NOT connect)
- **TX**
- **RX**
- **GND**

### Serial settings:
```
57600 baud
8N1
No flow control
```

You should get a U‑Boot prompt and full kernel boot logs.

---

## 🏁 Booting LEDE/OpenWrt via initramfs (temporary RAM boot)

This is the safest way to recover the device without risking a hard brick.

Standard Ip Config which i found:- 
Hootoo is 10.10.10.123
You set your pc to 10.10.10.3

-----------------
Press the number 2 key for tftp upload
-----------------
Have a Tftp server ready, Tftpd64 for windows or other

1. Interrupt U‑Boot 
2. Use TFTP to load an initramfs image:
   ```
   tftpboot 0x80000000 lede-ramips-rt305x-ht-tm02-initramfs-kernel.bin
   bootm 0x80000000
   ```

The device will boot LEDE entirely from RAM. ??? Well , that was the original idea, hope it works for you, but there is a second option which worked for me

---

## 🔥 Flashing the firmware permanently (raw MTD method)

Once booted into initramfs:

### Identify MTD layout:
```
cat /proc/mtd
```

You should see:
```
mtd3: 007b0000 "firmware"
```

### Flash the sysupgrade image directly:
```
mtd write /tmp/lede-ramips-rt305x-ht-tm02-squashfs-sysupgrade.bin firmware
reboot
```

The device now boots LEDE from flash.

--------------------------------------------------------------------
When things look dim.
---------------------------------------------------------------------
IF TFTP IMAGE FAILS:- 
RUN OPTION 4 WHICH IS TO USE THE RT5350# PROMPT... PRESS 4 TO INTERRUP U-BOOT AND GET THIS PROMPT
----
NOTE: the file names can be changed as you wish to shorten but you can copy and paste as well, dont stress.
---
RT5350 #
 
have tftp server ready, with file to be uploaded  (lede-ramips-rt305x-ht-tm02-initramfs-kernel.bin) 
lan ip address of eth to 10.10.10.3

RT5350 #  
---
setenv ipaddr 10.10.10.3
setenv serverip 10.10.10.123
---
tftpboot 0x80000000 lede-ramips-rt305x-ht-tm02-initramfs-kernel.bin
---
Once this file is uploaded (you should have seen activity on the screen and on the tftpd server screens,yo need to boot thee file. boot it :- 

bootm 0x80000000
---
Once booted it will stop and prompt you to press enter to activate the console, Press enter and you should see



BusyBox v1.25.1 () built-in shell (ash)

     _________
    /        /\      _    ___ ___  ___
   /  LE    /  \    | |  | __|   \| __|
  /    DE  /    \   | |__| _|| |) | _|
 /________/  LE  \  |____|___|___/|___|                      lede-project.org
 \        \   DE /
  \    LE  \    /  -----------------------------------------------------------
   \  DE    \  /    Reboot (17.01.7, r4030-6028f00df0)
    \________\/    -----------------------------------------------------------

=== WARNING! =====================================
There is no root password defined on this device!
Use the "passwd" command to set up a new password
in order to prevent unauthorized SSH logins.
--------------------------------------------------
root@LEDE:/#
 Now you have a minimal working system on the hootoo. The base Ip for the LEDE is 192.168.1.1
 
Change your ethernet adaptor to 192.168.1.10, basically anything other than 192.168.1.1

Next :- get the systems upgrade file ( lede-ramips-rt305x-ht-tm02-squashfs-sysupgrade.bin and put it in the tmp folder in LEDE , use winscp or whatever program you wish. 

LEDE is now at 192.168.1.1 , 

Username:- root and no password required, unless you have already set one.
Upload that file to LEDE lede-ramips-rt305x-ht-tm02-squashfs-sysupgrade.bin
Again files can be renamed shorter, but never forget the file extension.

Flash it from the lede shell  Back on the putty interface RT5350 # type:-
---
mtd -r write /tmp/tmp/lede-ramips-rt305x-ht-tm02-squashfs-sysupgrade.bin firmware
---
After a short time if all goes well, you sill see that you have a working LEDE router with version 17 loaded.

--------------------------------------------
## 🌐 Restoring network access

Default LEDE config puts **eth0 in LAN**, so WAN does not exist.  
Create a WAN interface:

```
uci set network.wan=interface
uci set network.wan.ifname='eth0'
uci set network.wan.proto='dhcp'
uci commit network
/etc/init.d/network restart
```
Unplug your ethernet to the device and reset the ethernet to dhcp, plug in the wan cable into the hootoo.
Now the device has internet access
----------------------------------------
## 📡 Enabling Wi‑Fi

Generate wireless config:
```
wifi config
```

Edit:
```
vi /etc/config/wireless
```

Enable radio:
```
option disabled '0'
```

Set AP mode:
```
option ssid 'HTTM02'
option encryption 'psk2'
option key 'yourpassword'
```

Apply:
```
wifi reload
```

---

## 🧹 Optimizing for AP‑only mode (recommended)

To reduce RAM usage and improve stability:

Disable IPv6:
```
opkg remove odhcpd-ipv6only kmod-ipv6
```

Disable firewall (AP‑only):
```
/etc/init.d/firewall stop
/etc/init.d/firewall disable
opkg remove firewall
```

Disable DHCP server (if main router handles DHCP):
```
/etc/init.d/dnsmasq stop
/etc/init.d/dnsmasq disable
opkg remove dnsmasq
```

Disable logging:
```
/etc/init.d/log stop
/etc/init.d/log disable
```

Install Luci Ui if you wish
opkg update
opkg install luci

Inside Luci you may check for other packages which remain, 
iPv6
ppp
ppoe
etc


After tuning, the device typically has:
- **10–12 MB free RAM**
- **rock‑solid uptime**
- **snappy LuCI performance**

---

## 📈 Expected Performance

Due to the RT5350F CPU, Wi‑Fi throughput is limited to:

- **25–30 Mbps** with WPA2  
- **30–35 Mbps** open Wi‑Fi  
- **40 Mbps** theoretical max (CPU‑bound)

This is normal and expected.

---

## 📝 Files Included in This Repository

- Serial logs  
- Working LEDE images  
- MTD layout reference  
- Example `/etc/config` files  
- Recovery scripts  
- Known‑good AP‑only configuration  

---

## ❤️ Why This Repo Exists

Because too many people have nearly given up on this tiny router.  
This guide exists so the next technician doesn’t have to fight through:

- missing wireless configs  
- broken vendor firmware  
- dead web UIs  
- confusing MTD layouts  
- outdated OpenWrt pages  
- and the fear of bricking the device permanently  

If this helped you, consider contributing improvements or sharing your own recovery story.


## 📚 Resources

### 🎥 Video Guide
[![Watch on YouTube](https://img.shields.io/badge/▶_Watch_on_YouTube-red?style=for-the-badge)](https://www.youtube.com/watch?v=H49qRZCimNA)

[![Hootoo Tripmate Nano OpenWRT Password Recovery](https://img.youtube.com/vi/H49qRZCimNA/0.jpg)](https://www.youtube.com/watch?v=H49qRZCimNA)

*Credit: Original recovery demonstration by the YouTube creator.  
Thanbk you AReResearch  @AReResearch
Linked here to help others attempting to unbrick or restore the HT‑TM02.*

---

### 📄 Documentation & Files
- Serial logs (coming soon)
- Working LEDE/OpenWrt images
- Known‑good `/etc/config` files
- MTD layout reference
- Recovery commands & notes

---

### 🔗 Useful Links
- OpenWrt Wiki — HooToo TripMate Nano  
  https://openwrt.org/toh/hootoo/tripmate-nano
  
- OpenWrt Forum — Debricking discussions  
  https://forum.openwrt.org/

