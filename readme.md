# PortaNas

# ** This is Work In Progress at the moment **

PortaNas is a list of instructions to getting a Raspberry Pi set up so that it can be used as portable storage. It can also be used if you just want a simple file server too. A Pi Zero, 1 or 2 aren't going to set any speed / performance records. (My Pi Zero achieves 1.4MB/s on Wifi writing direct to a class 10 high performance SD card), however a 3 or 4 will be much better at achieving higher speeds. There are many factors that will affect the speed however which is beyond the scope of this document.


## Getting Started
These instructions are based on using the Lite version of Rasbian (for a headless set-up) - other OS's may work or have their own quirks to work around.

**It is best carrying out these few steps connected to a monitor and keyboard.**

1. Obtain a copy of Rasbian (Lite) and use whatever method to write the image to the Pi's SD card. A good guide can be found [here](https://www.raspberrypi.org/documentation/installation/installing-images/)

2. Insert the SD card to the Pi, connect a monitor and also a keyboard. Boot the Pi and log in. The default username is `pi` and password is `raspberry`.  

3. The `raspi-config` tool should automatically load on first use, however if it doesn't, type `sudo raspi-config` and hit enter

4. [This page](https://www.raspberrypi.org/documentation/configuration/raspi-config.md) details how to use the `raspi-config` tool and where to adjust the next options. 

5. Change the password for user `Pi` and use a good password that isn't easily guessed but is memorable. If it helps, write it down somewhere safe - in an address book, a mobile phone contact etc. *It isn't good practice, but we are human after all!*  **This will be the administration account so please don't lose these details**. This option is found in the main menu under **Change user password**.

6. Change the hostname to ``portapi``. This can be anything within the parameters of the hostname spec (noted within the `raspi-config` tool). This document however will make reference to this hostname - just substitute your own for it). This option is found under **Advanced Options > Hostname**.

7. Adjust the memory split for the GPU (as it's a headless system - 16Mb for GPU will give the greatest amount of RAM for the system). This is under **Advanced Options > Memory Split**.

8. Enable SSH (see [here](https://www.raspberrypi.org/documentation/remote-access/ssh/) for details on the `raspi-config` method)

9. For a self contained system the SD card can be used as the usable storage space. For this set-up we will create a new partition that will be used for storage, by shrinking the Root partition to 8GiB and creating a new EXT4 partition in the remaining space. This can best be achieved through item 6 or 7 [here](https://elinux.org/RPi_Resize_Flash_Partitions#Manually_resizing_the_SD_card_on_Raspberry_Pi). *It is much easier to use Gparted on another system as this has a better visual representation of what is being done to avoid mistakes being made.*

10. The usable size of the storage partition can be used by reducing the reserved blocks on it (these are mainly used by the system if it runs out of space to keep things working, however the system partition will have it's own reserved blocks. Reduce the reserved blocks on this new EXT4 partition to zero using:

	`tune2fs -m0 /DEVICEPARTITION` 

	where `DEVICEPARTITION` is `/dev/mmc0blk3` or whatever the partition is identified as (it may be `/dev/sdb3` if it is done on another system. The partition can be identified using `sudo fdisk -l` and looking at the device column). **It has to be the partition and not just the whole block device.**
	
5. Ensure the Pi is connected to a network - if it's ethernet, then ensure the cable is in. If it's wifi - use the guide [here](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)

6. Reboot the Pi and either connect to it through SSH, or continue


