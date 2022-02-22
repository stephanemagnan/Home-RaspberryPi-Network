# Home-RaspberrryPi-Network
Steps to setup my Raspberry Pi-based home server
Experience before starting: Absolute beginner with Linux, some knowledge of RP, heard of docker.

Current Feature:
- Raspberry Pi 4B
- Raspberry OS Lite (requires a non-UI install)
- Boot OS from SD card
- All data saved to external SSD hardrive
- OpenMediaVault (OMV)
  - Docker
  - Portainer
    - Heimdall
    - SpeedTest Tracker
    - Pi-hole [Pending]
    - Unbound [Pending]
    - NextCloud [Pending]
  - SMB FILE SHARE


## INSTALL RASPBERRY PI OS:
Download and install the Raspberry Pi Imager: https://www.raspberrypi.com/software/
Insert micro SD in card reader
Open RPI
- Choose OS: Raspberry Pi OS Lite (32 bit) // Must not have desktop environment for use with OMV
- Choose Storage: Select ${micro SD card drive}
Enable advanced options: press CTRL+SHIFT+X
- Enable SSH with "Use password authentication"
- Set username and password
- [optional] Configure wifi //I recommend ethernet connection if possible
- Set locale settings
Save and [WRITE] to disk

Insert SD card into RP
[recommended] Plug into external monitor to monitor progress
Power on RP and wait for installation to complete

Reserve your IP address for the RP in your router software
- I won't go into detail here as router software varies significantly
- You need to set a static IP address for your RP so that 1) you know the SSH IP address, and 2) services such as pi-hole don't freak out. This is not the same as a static IP from your ISP. 
- I recommend using an easy to remember number such as 10.0.0.10 (if you are using the 10.0.0.X series of addresses) or 192.168.0.192 (if you are using the 192.168.0.X series of addresses). Make it something you will remember because you will use it a lot.

## PREPARE EXTERNAL HARDDRIVE
REFS:
-

I am using a SeaGate 1 TB external SSD harddrive. I am not using an external powered USB hub. RP appears to be providing sufficient power.
Before use, the SSD must be partitioned for use [if applicable].

I have the following partitions:
- 256 MB - FAT32 - boot
-  GB - EXT4 - root
- 2 GB - EXT4 - swap
- [remainder] - NTFS - storage

A few notes:
- I am still booting off of and using the micro SD card for the boot and root drives. I plan to eventually migrate those over to the external drive at some point.
- The SWAP drive is not required, but can be used to increase your avaiable RAM using your SSD. I have a 4 GB RAM RP, the max SWAPFILE size I could have was 2 GB.
- The main storage is recommended to be EXT4, however, I am using NTFS because I want to be able to plug my SSD into my windows computer to extract data if the RP crashes. There appear to be 3rd party tools that allow for a windows computer to read EXT4. I did not explore this option.
- The first three partitions are primary partitions. The NTFS partition is an extended partition (which allows for an extra layer of partitioning). Therefore, one extra sub-partition is required to "fill" this one.

### SET-UP
SSH into RPI Using your tool of choice. 
- I use PuTTY (download here: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
- I didn't use the hostname for SSH. I use the IP address of my RP. [SEE INSTALL RASPBERRY PI OS for notes on reserving a fixed IP for your RP]
- Log in using your RP info [SEE INSTALL RASPBERRY PI OS for setting username and password during setup]

## INSTALL OPENMEDIAVAULT
REFS:
-

To access OMV, type the IP address of the RP in the browser on a separate computer (for example 10.0.0.10).

## CONFIGURE SSD IN OPENMEDIAVAULT
REFS:
-

The external harddrive must be mounted in OMV before use. I created shared folders on my external harddrive for all of my applications and containers. 
Open OMV (type IP of RP in the browser).

Mount the NTFS partition
- Storage> Disk> File Systems> (+)> Mount > [SELECT DRIVE PARTITION. ex: dev/sda3 NTFS XYZ GB]

To create new shared folders
- Storage> Shared Folders> (+)> [ADD NAME - I use application name unless I have a reason to use something else]
                                [SELECT FILE SYSTEM - from previous step]
                                [RELATIVE PATH - for all apps, I created folders in var/lib/${APP NAME} to keep everything isolated]

## INSTALL DOCKER (OMV)
REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: "docker": var/lib/docker/

## INSTALL PORTAINER (OMV)
REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: "portainer": var/lib/portainer/

## INSTALL HEIMDALL (PORTAINER)
REFS:
- https://hub.docker.com/r/linuxserver/heimdall


Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: "heimdall": var/lib/heimdall/

## INSTALL SPEEDTEST TRACKER (PORTAINER)
REFS:
- https://github.com/henrywhitaker3/Speedtest-Tracker


Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: "speedtesttracker": var/lib/speedtesttracker/

## INSTALL PI-HOLE (PORTAINER)
REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: var/lib//
Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: var/lib//

## INSTALL UNBOUND (PORTAINER)
REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: var/lib//

## INSTALL NEXTCLOUD (PORTAINER)
REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: var/lib//

## CONFIGURE SMB FILE SHARE (OMV)
REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: "NASa": shared/NASa public/
Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: "NASb": shared/NASb private/

