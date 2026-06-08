# Portable Homelab
A portable 3D-printed homelab built with Raspberry Pis, containing:

- [OpenWRT Router](#openwrt-router)
- [5-Port Switch](#5-port-gigabit-switch)
- [Network Attached Storage](#network-attached-storage)
- [Docker](#docker)

## 3D Printed Rack

I used the files for [**Mini Lab Rax 5" Server Rack - 5U**](https://makerworld.com/en/models/2499872-mini-lab-rax-5-server-rack-5u) by mkelements - a 1/2 scale version of their 10" LabRax system. They were printed on a Bambu P1S using PETG.

There are several other good sets of files, such as the 6" [**Microlab - Mini Modular Home Server Rack**](https://makerworld.com/en/models/1062225-microlab-mini-modular-home-server-rack).

## Server Components

### OpenWRT Router

### 5-Port Gigabit Switch

A TP-Link 5-Port Gigabit desktop switch (LS1005G). The switch has to be removed from the case to fit properly, but it pops out easily. For such a small system I figured there wasn't a need for one with VLAN capabilities. 

### Network Attached Storage

A small NAS running Samba for file sharing and storing the Docker containers.

- Raspberry Pi 5 (1GB)
- Pi 5 Active Cooler
- Pimoroni NVMe Base
- 256GB NVMe drive
- Elegoo 0.96" OLED Display

**Setup**

Connect the NVMe Base (after installing the drive), and boot using Raspberry Pi OS Lite (64-bit). Run the usual updates:

```
sudo apt-get update
sudo apt-get upgrade
```

After verifying that the Pi can see the NVMe drive, we're going to configure the system to boot from NVMe and get rid of the SD card. Clone the SD card to the NVMe drive by typing:

`sudo dd if=/dev/mmcblk0 of=/dev/nvme0n1 status=progress`

Once that is complete, set the Pi to boot from the NVMe. Start the Raspberry Pi configuration tool:

`sudo raspi-config`

Choose Advanced Options, Boot Order, NVMe/USB Boot. Click No when asked to reboot. Shut down the Pi with:

`sudo shutdown -h now`

Unplug the Pi, remove the SD card, plug it back in, and reboot. You should now be running off the NVMe.

If you want to add the mini OLED display, [follow the instructions here](https://the-diy-life.com/add-an-oled-stats-display-to-raspberry-pi-os-bookworm/).

Next, it's time to configure Samba for file sharing. First, install Samba:

`sudo apt-get install samba samba-common-bin`

Create the shared folder (this assumes the username is 'pi'):

`mkdir /home/pi/shared`

Modify Samba's configuration file to tell it where the folder is located and how to handle access:

`sudo nano /etc/samba/smb.conf`

At the end of the file, add:

```
[PiShare]
path=/home/pi/shared
writeable = yes
browseable = yes
public = no
valid users = pi
```

Save the configuration file and exit the editor. Next, add the pi user to Samba's user database:

`sudo smbpasswd -a pi`

And restart the Samba server:

`sudo systemctl restart smbd`

You should now be able to connect to the server using \\{IP_ADDRESS}\PiShare

### Docker
