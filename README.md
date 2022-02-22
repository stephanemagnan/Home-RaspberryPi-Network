# Home-RaspberryPi-Network
Steps to setup my Raspberry Pi-based home server.

Limitations:
- This guide was developed after I finished most of the setup, so it is possible that I have missed a small step or two, however, the majority should be correct as I have verified it to the extent possible.

Who is this for?
- Anyone trying to set up their RP for home network/server use.
- You don't need to know much to follow these steps. My experience before starting: Absolute beginner with Linux, some knowledge of RP, heard of docker.

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
REFERENCES:
- https://www.instructables.com/Configuring-the-Raspberry-Pi-ethernet-port-for-rem/

STEPS:
1. Download and install the Raspberry Pi Imager: https://www.raspberrypi.com/software/
2. Insert micro SD in card reader
3. Open RPI
   - Choose OS: Raspberry Pi OS Lite (32 bit) // Must not have desktop environment for use with OMV
   - Choose Storage: Select ${micro SD card drive}
4. Enable advanced options: press CTRL+SHIFT+X
  - Enable SSH with "Use password authentication"
  - Set username and password
  - [optional] Configure wifi //I recommend ethernet connection if possible
  - Set locale settings
  - Save settings
5. WRITE the image to disk
6. Boot RP
   - [recommended] Plug into external monitor to monitor progress
   - Insert SD card with OS image into RP
   - Power on RP 
   - Wait for installation to complete
7. Update IP address for RP
   - You need to set a static IP address for your RP so that 1) you know the SSH IP address, and 2) services such as pi-hole don't freak out. This is not the same as a static IP from your ISP. 
   - You can reserve your IP address for the RP in your router software. I won't go into detail here as router software varies significantly
     - [recommended] reserve an easy to remember IP address such as 10.0.0.10 (if you are using the 10.0.0.X series of addresses) or 192.168.0.192 (if you are using the 192.168.0.X series of addresses). Make it something you will remember because you will use it a lot.
     - [otherwise] use the automatically assigned IP for your RP, but change to static so that it does not change
   - You have two options for sdetermining the IP address assigned to the RP. 1) Log into your router to identify the IP address of the RP. Or, [recommended] use a monitor + keyboard connected to your RP to run the ... ifconfig ...  command to view the IP address assigned to the RP.
   - You can probably perform the next step using 8a) SSH but I expect you will lose connection after modifying the settings especially if the IP address is changed. You would then need to reconnect using SSH and the new IP. I have not tested this method and used my [recommended] 8b) monitor + keyboard connected to the RP.
8a. SSH into RPI Using your tool of choice. 
    - I use PuTTY (download here: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
    - I didn't use the hostname for SSH. I use the IP address of my RP. [SEE INSTALL RASPBERRY PI OS for notes on reserving a fixed IP for your RP]
    - Log in using your RP info [SEE INSTALL RASPBERRY PI OS for setting username and password during setup]
    - Follow steps in 8b, reconnecting to RP through SSH if needed after any IP config changes.
8b. Type the following commands into the RP command window: [from instructables: Configuring the Raspberry Pi Ethernet Port With a Static IP Address - author: ZRob314]
    - View network status:
...
ifconfig
...
    - Backup network settings 
....    
sudo cp /etc/dhcpcd.conf /etc/dhcdcp.backup
...

    - Edit network settings to have fixed IP address
...    
sudo nano /etc/dhcpcd.conf
...
    - Make the following changes to start of the file:
...
interface eth0
static ip_address=10.0.0.10/24 
...
8c. I recently discovered that you may be able to change these settings from the OMV menu, and the dhcpcd.conf file is moved/changed by the OMV install.


## PREPARE EXTERNAL HARDDRIVE

REFS:
- https://forums.raspberrypi.com/viewtopic.php?t=261455
- https://tldp.org/HOWTO/Partition/fdisk_partitioning.html

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
- The main storage is recommended to be EXT4, however, I am using NTFS because I want to be able to plug my SSD into my windows computer to extract data if the RP crashes. There appear to be 3rd party tools that allow for a windows computer to read EXT4 (it is not read natively). I did not explore this option.
- The first three partitions are primary partitions. The NTFS partition is an extended partition (which allows for an extra layer of partitioning). Therefore, one extra sub-partition is required to "fill" this one.

### SET-UP
8. SSH into RPI Using your tool of choice. 
- I use PuTTY (download here: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
- I didn't use the hostname for SSH. I use the IP address of my RP. [SEE INSTALL RASPBERRY PI OS for notes on reserving a fixed IP for your RP]
- Log in using your RP info [SEE INSTALL RASPBERRY PI OS for setting username and password during setup]



## INSTALL OPENMEDIAVAULT

REFS:
-

To access OMV, type the IP address of the RP in the browser on a separate computer:
- http://10.0.0.10

## CONFIGURE SSD IN OPENMEDIAVAULT

REFS:
-

The external harddrive must be mounted in OMV before use. I created shared folders on my external harddrive for all of my applications and containers. 

Open OMV (type IP of RP in the browser).

Mount the NTFS partition
- Storage> Disk> File Systems> (+)> Mount > 
  - [SELECT DRIVE PARTITION. ex: dev/sda3 NTFS XYZ GB]

To create new shared folders
- Storage> Shared Folders> (+)> 
  - [ADD NAME - I use application name unless I have a reason to use something else]
  - [SELECT FILE SYSTEM - from previous step]
  - [RELATIVE PATH - for all apps, I created folders in var/lib/${APP NAME} to keep everything isolated]

## INSTALL DOCKER (OMV)

REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
- "docker": var/lib/docker/

## INSTALL PORTAINER (OMV)

REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
- "portainer": var/lib/portainer/


To access PORTAINER, type the IP address of the RP + port for the service in the browser on a separate computer:
- http://10.0.0.10:9000

## INSTALL HEIMDALL (PORTAINER)

REFS:
- https://hub.docker.com/r/linuxserver/heimdall


Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
- "heimdall": var/lib/heimdall/


To access HEIMDALL, type the IP address of the RP + port for the service in the browser on a separate computer:
- http://10.0.0.10:6941

Steps for adding a new service to HEIMDALL:
- Go to HIEMDALL service
- Select (..) button to view services as list
- Click add service, populating all fields (including url with http://10.0.0.10:PORT)
- Click Save
- Select (<- ->) button to rearrange buttons on main page

Add OPENMEDIAVAULT and PORTAINER to HEIMDALL [see steps for adding new service to HEIMDALL in INSTALL HEIMDALL (PORTAINER)]

## INSTALL SPEEDTEST TRACKER (PORTAINER)

REFS:
- https://github.com/henrywhitaker3/Speedtest-Tracker
- https://www.youtube.com/watch?v=nMNlkDPohcc


Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
- "speedtesttracker": var/lib/speedtesttracker/




To access SPEEDTEST TRACKER, type the IP address of the RP + port for the service in the browser on a separate computer:
- http://10.0.0.10:8080

Add SPEEDTEST TRACKER to HEIMDALL [see steps for adding new service to HEIMDALL in INSTALL HEIMDALL (PORTAINER)]

## INSTALL PI-HOLE (PORTAINER)

REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
- var/lib//
- var/lib//


To access PI-HOLE, type the IP address of the RP + port for the service in the browser on a separate computer:
- http://10.0.0.10:

Add PI-HOLE to HEIMDALL [see steps for adding new service to HEIMDALL in INSTALL HEIMDALL (PORTAINER)]

## INSTALL UNBOUND (PORTAINER)

REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: var/lib//

## INSTALL NEXTCLOUD (PORTAINER)

REFS:
-

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: var/lib//

Add NEXCLOUD to HEIMDALL [see steps for adding new service to HEIMDALL in INSTALL HEIMDALL (PORTAINER)]

## CONFIGURE SMB FILE SHARE (OMV)
REFS:
- https://www.reddit.com/r/DataHoarder/comments/3xt15i/whats_the_name_of_your_nas/
- https://dannyda.com/2019/07/17/how-to-create-smb-cifs-windows-share-in-open-media-vault-omv/

Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
- "NASa": shared/NASa public/
- "NASb": shared/NASb private/

