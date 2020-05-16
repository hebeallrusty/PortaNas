# PortaNas

**The instructions will enable the Pi to be accessed through SFTP and Samba, however the instructions for creating a wireless hotspot are not yet completed or tested**

PortaNas is a list of instructions to getting a Raspberry Pi set up so that it can be used as portable storage. It can also be used if you just want a simple file server too. A Pi Zero, 1 or 2 aren't going to set any speed / performance records. (My Pi Zero achieves 1.4MB/s on Wifi writing direct to a class 10 high performance SD card), however a 3 or 4 will be much better at achieving higher speeds. There are many factors that will affect the speed however which is beyond the scope of this document.


## 1. Getting Started
These instructions are based on using the Lite version of Rasbian (for a headless set-up) - other OS's may work or have their own quirks to work around.

**It is best carrying out these few steps connected to a monitor and keyboard.**

1. Obtain a copy of Rasbian (Lite) and use whatever method to write the image to the Pi's SD card. A good guide can be found [here](https://www.raspberrypi.org/documentation/installation/installing-images/)

2. Insert the SD card to the Pi, connect a monitor and also a keyboard. Boot the Pi and log in. The default username is `pi` and password is `raspberry`.  

3. The `raspi-config` tool should automatically load on first use, however if it doesn't, type `sudo raspi-config` and hit enter

4. [This page](https://www.raspberrypi.org/documentation/configuration/raspi-config.md) details how to use the `raspi-config` tool and where to adjust the next options. 

5. Change the password for user `pi` and use a good password that isn't easily guessed but is memorable. If it helps, write it down somewhere safe - in an address book, a mobile phone contact etc. *It isn't good practice, but we are human after all!*  **This will be the administration account so please don't lose these details**. This option is found in the main menu under **Change user password**.

6. Change the hostname to ``portanas``. This can be anything within the parameters of the hostname spec (noted within the `raspi-config` tool). This document however will make reference to this hostname - just substitute your own for it). This option is found under **Advanced Options > Hostname**.

7. Adjust the memory split for the GPU (as it's a headless system - 16Mb for GPU will give the greatest amount of RAM for the system). This is under **Advanced Options > Memory Split**.

8. Enable SSH (see [here](https://www.raspberrypi.org/documentation/remote-access/ssh/) for details on the `raspi-config` method)

9. For a self contained system the SD card can be used as the usable storage space. For this set-up we will create a new partition that will be used for storage, by shrinking the Root partition to 8GiB and creating a new EXT4 partition in the remaining space. This can be achieved through item 6 or 7 [here](https://elinux.org/RPi_Resize_Flash_Partitions#Manually_resizing_the_SD_card_on_Raspberry_Pi). *It is much easier however to use Gparted on another system as this has a better visual representation of what is being done to avoid mistakes being made.*

10. The usable size of the storage partition can be increased by reducing the reserved blocks on it (these are mainly used by the system if it runs out of space to keep things working, however the system partition will have it's own reserved blocks). Reduce the reserved blocks on this new EXT4 partition to zero using:

	`tune2fs -m0 /DEVICEPARTITION` 

	where `DEVICEPARTITION` is `/dev/mmc0blkp3` or whatever the partition is identified as (it may be `/dev/sdb3` if it is done on another system. The partition can be identified using `sudo fdisk -l` and looking at the device column). **It has to be the partition and not just the whole block device.**
	
11. Ensure the Pi is connected to a network - if it's ethernet, then ensure the cable is in. If it's wifi - use the guide [here](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)

12. Update the Pi and install install git by issuing the following:
	
	`sudo apt-get update`
	
	`sudo apt-get install git`
	
	follow the onscreen text, and control will be return to the prompt at the end

13. Install Screen which allows the Pi's terminal to be duplexed or returned to if the current terminal session is disconnected:

	`sudo apt-get install screen`
	
	again follow the onscreen text until control is returned to the prompt

14. Shut down the Pi by issuing:

	`sudo shutdown -h now`
	
	The Pi will shut down after a few seconds

## 2. Connecting over SSH

Now we have the bones of a working system. We need to set up the storage so that we can start saving files to it, however, as the intention is to have a headless system (i.e we don't want it connected to a keyboard and monitor all the time) first we need to connect to it from another terminal.

We can either continue this section with the Pi connected to the monitor and keyboard just in case something goes wrong or doesn't work as expected. 

We should have a working SSH config that will allow us to connect remotely from another system that has SSH installed (Windows users will need something like PuTTY; Linux and Mac users should have it installed by default)

As we have set up the Pi with a memorable hostname, this can now be used by other systems to *easily* connect to it without knowing it's IP address (in practice this could be complicated). 

We would use `hostname.local` where hostname is the one set up earlier (`portanas.local` in our case). This all depends if a mDNS service is installed on your system (avahi in the case of Linux; Windows will need the bonjour service installed which can be extracted from the iTunes installer - just download iTunes, and instead of installing, use 7zip to open the iTunes installer, extract the item marked bonjour.msi and run that file. Newer versions of Windows 10 also requires some registry tweaking to allow it to use the bonjour service correctly. [Here](https://superuser.com/questions/1330027/how-to-enable-mdns-on-windows-10-build-17134) is a guide to adjust the registry).

To test if your system can use the `hostname.local` method, ensure the Pi is connected to your network first by typing in (on the Pi):

`ping 192.168.1.1`
	
or whatever your router's IP address is (it could be `192.168.0.1` - consult with your router's documents to find this out as some manufacturers put the router on different subnets. You can also substitute the ping command with `ping google.com` or any other website which should only be successful if it goes through your router). Ensure the response is good before proceeding (and that you are not getting anything that says unreachable)
	
Now on your system, open the command line and type in:
	
`ping portanas.local`
	
If you get anything that says unreachable, then the problem lies on your system and not the Pi. If resolving this is beyond your capabilities, then the IP address of the Pi can be found by using a network scanner app on your phone such as Fing which is available for both iOS and Android. This will be the way to find the IP address when there is no monitor attached (otherwise it would be trivial to find the IP address by asking the Pi for it directly)
	
In the command line, connect to the Pi over SSH using the following command:
	
`ssh pi@portanas.local`

or if using PuTTY on Windows 

`putty.exe -ssh pi@portanas.local`
	
if you have the IP address instead, substitute the `portanas.local` above with the IP address you have obtained.
	
Accept all the notifications that are generated, and you should then be left with a command prompt from the Pi which will look like the following:
	
`pi@portanas: ~$`
	
Once you are at this prompt, then all commands that are entered are carried out on the Pi.
	
## 3. Setting up Storage

We need to add a folder to the system so that we can access the storage partition created earlier. To do this enter the following command:

`sudo mkdir /media/StorageSD`

We can use the `/media` folder later if we want to add any other storage drives to the system
	
Next we need to find out the unique reference of the Storage partition created earlier. We can do this by issuing the following command:

`sudo blkid`
	
This will display all the information relating to the storage devices connected to the Pi - an example is given below:

```bash
pi@portanas:~ $ sudo blkid
/dev/mmcblk0p1: LABEL_FATBOOT="boot" LABEL="boot" UUID="4BBD-D3E7" TYPE="vfat" PARTUUID="738a4d67-01"
/dev/mmcblk0p2: LABEL="rootfs" UUID="45e99191-771b-4e12-a526-0779148892cb" TYPE="ext4" PARTUUID="738a4d67-02"
/dev/mmcblk0p3: LABEL="STORAGE" UUID="55d7a201-0b23-4a4d-a77a-e3b79f425e73" TYPE="ext4" PARTUUID="738a4d67-03"
/dev/mmcblk0: PTUUID="738a4d67" PTTYPE="dos"
```
	
In the above case we need to know the UUID of `/dev/mmcblk0p3` which is `55d7a....425e73`. This is the unique reference to the Storage partition that was created earlier and will be different on your Pi. The reference is what we need to inform the Pi to mount on every boot and needs to be inserted into the file that tells the Pi how to mount partitions. It also tells the Pi which folder to mount the partition to as well. The file is found at `/etc/fstab`

We can either write the reference down carefully; highlight over the text, right click and select copy, then paste into the file when it is open; or append the reference directly into the file

To append the reference directly into the file we would issue the following command:

`sudo blkid -s UUID -o value /dev/mmcblk0p3 | sudo tee -a /etc/fstab`

we now need to edit the file and add in some extra options

`sudo nano /etc/fstab`

We need to put `UUID=` before the reference and add in some extra options. Make the line look like the following (again your reference will be different to the below):

```bash
UUID=55d7a201-0b23-4a4d-a77a-e3b79f425e73 /media/StorageSD ext4 defaults,noatime 0 2
```

+ The first entry is the partition reference (`UUID=55d7a201-0b23-4a4d-a77a-e3b79f425e73`). 
+ The second is which folder to mount it to (`/media/StorageSD`). 
+ The third entry is the partition type (`ext4`)
+ The fourth entry is the mount options (`defaults,noatime`) which in this case is the default options, plus we don't want the system to write when a file was accessed unless it was actually opened.  Otherwise just listing the contents of a folder will alter the accessed time incurring performance penalties and also adding potentially unnecessary writes to the SD card.
+ The fifth and sixth options tells the system whether to back up the partition (0) and to check the partition with fsck after the root partition (2)

The file should look like the following (references will be different on your system):
```bash
proc            /proc           proc    defaults          0       0
PARTUUID=738a4d67-01  /boot           vfat    defaults          0       2
PARTUUID=738a4d67-02  /               ext4    defaults,noatime  0       1
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that
UUID=55d7a201-0b23-4a4d-a77a-e3b79f425e73 /media/StorageSD ext4 defaults,noatime 0 2
```

Save and exit the file by entering `CTRL+X` then `Y` then `Enter`

This will close nano and return control back to the command line.

Test that the mount options work by issuing:

`sudo mount -a`

There should be no feedback if the command was successful, however if it was unsuccessful, then an error will be displayed. It should be noted that if there is an error and the Pi is needing to be rebooted **do not reboot the Pi without first commenting out the line that was added in `/etc/fstab` (put a `#` at the begining of the line, then `CTRL+X` then `Y` then `Enter`) otherwise the Pi will boot into emergency mode. See [here](https://www.raspberrypi.org/forums/viewtopic.php?t=211349) on how to fix in this case.**. The `nofail` option can be added to options above (use a comma to add it to the fourth option in `/etc/fstab` for that line). This will allow the system to boot if the mount has a problem, however as the Storage partition resides on the same disk as the operating system, it really should flag if there is an issue to prevent further data loss.

To verify that the mount was successful enter the following command which details how much space is free on the Pi:

`df -h`

There should be an entry which the last column (`Mounted on`) will be `/media/StorageSD` and the size should match what you are expecting.

## 4. Logging to RAM

**THIS SECTION IS OPTIONAL, HOWEVER PLEASE READ AND DECIDE IF YOU WANT TO FOLLOW IT**

An SD card has a limited number of writes before potential failure. Wear levelling is a technology built into them to spread writes as even as possible accross the whole card. The idea is that it prevents sections of the card getting more wear than others which reduces the chance of the card failing early. The bigger the SD card however the greater the area that the writes can be spread over, so theoretically it should be less prone to an early failure.

That being said, any way to reduce writes to the disk can potentially increase the life of the card. A card can however just be defective and fail early anyway - sometimes there is just no reasoning for it.

Linux likes to write log files to assist with debuging, auditing and general information, which could reduce the life of the SD card.

These logs can be written instead to RAM instead of the SD card to increase the life of the card, however be warned that when the Pi reboots, these logs are lost to the ether and are irretrevable. It can also make problem solving more difficult if the Pi randomly crashes or goes offline.

If you would like to log to RAM then proceed, otherwise skip this section and move on to the next one

*The original instructions for log2ram have been taken from [here](https://mcuoneclipse.com/2019/04/01/log2ram-extending-sd-card-lifetime-for-raspberry-pi-lorawan-gateway/).*

Make a new folder to store the install files for Log2Ram by issuing the following:

`mkdir /home/pi/src`

Move into the folder:

`cd /home/pi/src`

Clone the installation files:

`git clone https://github.com/azlux/log2ram.git`

Move into the folder that has now downloaded, make the installer executable and run it:

`cd ./log2ram`
 
`sudo chmod +x ./install.sh`

`sudo ./install.sh`

Following the onscreen text until control is returned to the prompt. The Pi needs to be rebooted to begin logging to ram. Issue the following:

`sudo reboot`

When the Pi has rebooted, check if the script has been successful by issuing the following:

`df -h`

The presence of a log2ram entry indicates that the process has been successful:

```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       7.9G  1.5G  6.2G  19% /
devtmpfs        236M     0  236M   0% /dev
tmpfs           241M     0  241M   0% /dev/shm
tmpfs           241M   20M  221M   9% /run
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           241M     0  241M   0% /sys/fs/cgroup
/dev/mmcblk0p3  109G  5.7G  104G   6% /media/StorageSD
/dev/mmcblk0p1  253M   53M  200M  21% /boot
log2ram          40M  1.4M   38M   1% /var/log
tmpfs            48M     0   48M   0% /run/user/1000
```

## 5. Installing automatic updates

To improve the security of the Pi, it is wise to ensure that updates are done regularly. 

As the system will be headless, and the only interaction will be to copy files, it may be tedious to SSH into it, type in a few commands, wait until they complete, then reboot and log out.

Therefore, it would be wise to automate these tasks so that it is almost a fit and forget solution. We can install the unattended upgrades package to carry out this task. 

On the command line, enter the following:

`sudo apt-get install unattended-upgrades`

This will install the package required. On Raspbian Buster, the package won't install the right updates as noted [here](https://raspberrypi.stackexchange.com/questions/38931/how-do-i-set-my-raspberry-pi-to-automatically-update-upgrade). We need to make an adjustment so that it pulls in all the security packages. We need to run the following command to fix it:

```bash
echo 'Unattended-Upgrade::Origins-Pattern {
//      Fix missing Rasbian sources.
        "origin=Debian,codename=${distro_codename},label=Debian";
        "origin=Debian,codename=${distro_codename},label=Debian-Security";
        "origin=Raspbian,codename=${distro_codename},label=Raspbian";
        "origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";
};' | sudo tee /etc/apt/apt.conf.d/51unattended-upgrades-raspbian
```

The above command adds additional sources for the unattended upgrades package to look at and install. There should be no feedback if the above was successful.

It would be wise to reboot the Pi at this stage to ensure that the package is executed correctly:

`sudo reboot`

## 6. Adding a lesser privileged user

It is also wise to add in another user that doesn't have administrative access to the whole system. Ideally every user that wants access to the system should have their own account, however this may become impractical if it is the occasional family member that needs access to store or retrieve something from the storage system. You wouldn't want to give them the master key would you? 

Likewise you shouldn't use the highest privileged account as an everyday account - even God had a day off on the Sabbath! (it's also really bad from a security point of view to use the admin account for trivial tasks)

This section will set up a general user that has no security privileges, however later on, additional steps will be detailed on adding extra users for a more permanent solution if multiple users are required.

To add another user enter the following on the command line:

`sudo adduser portanas`

The Pi will now prompt you to enter a password for this account and some other details. Only the password is manditory.

On non Debian based system - the `adduser` command is `useradd`. Both commands will work on Debian, but `adduser` does some extra magic behind the scenes and ensures that things will work correctly.

Also the username we have given is `portanas`. Any username (within reason) can be used instead - here `portanas` is used as it is generic to the system, however `frank`, `cloe` or `majorhaxxor` are all equally valid.

The user is added without any special privileges and cannot elevate their permissions such as the user `pi`. The user `portanas` is not a member of any group and therefore cannot access or change anything outside of their own `home` directory. We will however need to allow them access to the storage on the Pi through group permissions.

We will add a group that will assist with the permissions of files that are stored in the storage folder. For a single user, it is not necessarily a problem, however with multiple users, we need to have a bit of sanity about files in shared folders.

`sudo groupadd storage`

We now need to add the users of the Pi to this group:

`sudo usermod -a -G storage pi`

`sudo usermod -a -G storage portanas`

ensure you replace `portanas` above with whatever the username was that you chose earlier.

Now both `pi` and `portanas` are within the `storage` group.

We now need to adjust write and owner permissions of the `/media/StorageSD` folder so that both users are allowed to write to it. First we will change the owner of the folder as follows:

`sudo chown pi:storage -R /media/StorageSD`

This will now make the `StorageSD` folder owned by the user `pi` and the group `storage`. As both `pi` and `portanas` are part of the `storage` group, both users should now be able to access the folder, however we need to alter the permissions to ensure that this is the case. Note that the `-R` option means recursive, so it will go through all the subfolders altering the owner. If the folder is blank, this should be fine (as in the case of the remaining space on the SD card that we have set up above), but if you are following this guide with an external hard drive that has been used on another system and it already has files etc on it, it would be worth double checking that you are happy for the file owner and group to change. The command can be run without the `-R` option, however any sub-folders already in existance may no longer be accessible in this system.

To change the file permissions we will issue the following command:

`sudo chmod 2770 -R /media/StorageSD`

Again if there are existing files on the storage folder, the `-R` option will change all the files and folders to `2770` permissions which may or may not be acceptable.

Here `2---` means that newly created files, folders and subfolders will inherit the group ownership, meaning that other members of the group can access them. Without the `2---` in the permission, a file or folder could be created that no-one else could access or delete. This may in fact be what is required, however this system is designed as a portable accessible storage solution, and this can be changed if this behaviour is not desirable. 

Permissions `770` gives the owner and group of the file/folder read, write and execute permissions, whilst anyone else (i.e someone that isn't the owner, or within the `storage` group) will have no permissions at all.

## 7. Install Samba

Samba is a package that allows Windows to natively talk to the Pi and access the remote storage as if it were a Windows machine. It is also useful on Linux too as some distributions show the Windows Network in the file manager and allow you to connect through it without having any technical knowledge of what IP address the Pi is located at, or what folder can be accessed.

It should be noted at this point that the system can already be accessed through SCP or SFTP by virtue of having SSH working correctly and the storage folder correctly permissed.

Install samba as follows:

`sudo apt-get install samba`

Accept all the defaults at this stage. We need to add the users on the Pi into Samba otherwise we will not be able to connect even though the users exist on the Pi. To do this we will issue the following:

`sudo smbpasswd -a portanas`

**It is important that you enter the same password as the user account for `portanas`**. This prevents credential mismatches as password changes can propogate through Samba to the user account on the Pi. This can be altered through the configuration files, however it is not dealt with here.

Also we are not adding the user `pi` as a valid Samba user as that account is more privileged than the lesser account we have created.

We now need to modify the Samba config file to allow the shared folder to be accessible.

To do this we issue:

`sudo nano /etc/samba/smb.conf`

This will bring up the config file to edit. Within this file there will be a few things to change and add.

We need to add our storage folder at the very end of the file as follows:

```bash
[StorageSD]
	path = /media/StorageSD
	read only = no
	force create mode = 0660
	force directory mode = 2770
	force group = storage
```

we also need to comment out everything from:

```bash
[homes]
	comment = Home Directories
	...
	...
```

up to our share definition we have just created above. To comment out the line, we add a `#` in front of the line.

Without commenting out these sections, when browsing the network share, the user will be presented with folders that are not related to the share we have created. There is a folder named after their username which will link into their home directory on the Pi. As the Pi has been given 8GiB of storage for the whole system - it is possible that this space could be accidentally filled with files and deprive the system of space for updates and additional software. There are also folders for print drivers too which is not necessary.

Save and exit the file (`CTRL + X` then `Y`, then `Enter`)

We now need to re-load Samba so that it reads these changes by issing the following:

`sudo service smbd restart`

## 8. Accessing the shares

There are two ways now that the shares can be accessed - through SFTP (not to be confused with FTPS) and through Samba. There are many differing ways in which the shares can be accessed depending on the app, system etc which is beyond the scope of this document, however some hints will be given to assist with getting the connection up and running.

On a system that supports SFTP, you can just point it to the address `portanas.local` and the folder `/media` which will give you access to all shares available (if more are added later). This is assuming your system can resolve `portanas.local` to the IP address (iOS and Mac can; Linux depends on the distribution). If it can't, then point the app to the IP address of the Pi instead (a network probing app like Fing can help). There are a multitude of apps available, so it is just a case of having a look at the documents to see how the details are entered.

In Gnome 3 for example, click on `Other Locations` in the file manager and enter the following into the `Connect to Server` section: `sftp:portanas.local/media`, click `Connect` then enter the user details of the user on the Pi (username `portanas` (or what ever was set up earlier) NOT `pi`)

On a system that supports Windows Shares - the share may be automatically found by browsing the network. If not, the share can be found through the hostname alone preceeding it with two `\\`. In our case, this would be `\\portanas` 

Some apps that access the Windows Share require the Windows shared folder name to be added at the end of the addres which in this case would be `StorageSD`.

It should also be mentioned that Samba does not do SMB1 (the original insecure version) of Windows Sharing, and the minimum supported is SMB2. Ensure that your app is compatible with SMB2 as otherwise you will not be able to connect.

## 9. Adding extra users

Adding extra users is fairly straightforward and can be achieved through the following:

1. Log into the Pi with the `pi` user account

2. Enter the following command to create a new user:

	`sudo adduser <username>`
	
	where `<username>` is the username that is required.
	
3. Add them to the `storage` group:
	
	`sudo usermod -a -G storage <username>`
	
4. Add them into Samba:
	
	`sudo smbpasswd -a <username>`
	
	and use the same password that was supplied when adding the user account
	
5. Restart the Samba service:

	`sudo service smbd restart`
	
## 10. Turning the system into a truly portable system

This section will be filled in when I have carried out and tested this myself, however the idea is that by using `hostapd` the Pi can be made into a hotspot that can be connected to through wifi directly. This would be useful if you are out and about and need some portable storage and are without a computer to reconfigure the system to connect to a different network. Think of it as a wireless harddrive.



	



