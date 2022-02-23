# Home-RaspberryPi-Network
##Steps to setup my Raspberry Pi-based home server. 

WELCOME:
I created this guide because I couldn't find one source with the all the steps describing the setup and integration of all the components I wanted in my home RP network. From my research, I concluded that the set-up I wanted it pretty popular with lots of home server RP users having a combination of the components I installed. It may be easy for more experienced RP users to piece together the requirements from all the different sources. Not being particularly experienced, I wanted to record all my steps for anyone else to use or to make my life easier if my RP crashes and I have to set everything up again. There are very good websites and YouTube videos from a variety of really good creators that describe each component individually but I couldn't find anything with all of the steps. I needed to modify some of these guides to make everything work with my system. I added all the sources I could remember consulting as references but may have missed some. 

WHO IS THIS FOR?
- Anyone trying to set up their RP for home network/server use.
- You don't need to know much to follow these steps. My experience before starting: Absolute beginner with Linux, some knowledge of RP, heard of docker, significant programming experience in other languages.

CURRENT FEATURES:
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

LIMITATIONS:
- This guide was developed after I finished most of the setup, so it is possible that I have missed a small step or two, however, the majority should be correct as I have verified it to the extent possible.

FEEDBACK:
- I am far from an expert on any of the topics in this guide. If you notice incorrect information or have problems with any of the steps, feel free to open issues. Full disclosure, if it is not working, I may not have any idea how to fix the issues. You might be better of searching onlien forums for answers to your specific problem. If this happens, please let me know if something needs to be updated in the guide.


## INSTALL RASPBERRY PI OS:
REFERENCES:
- https://www.instructables.com/Configuring-the-Raspberry-Pi-ethernet-port-for-rem/
- https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up/2
- https://jamesjdavis.medium.com/how-to-update-raspberry-pi-just-follow-these-easy-steps-ac507cf70238

SUMMARY:
Install Raspian OS Lite on RP, boot RP and modify IP parameters to prepare for future use

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
   - Perform typical RP initialization functions:
...
sudo apt update
sudo apt upgrade
sudo reboot -h now
...
7. Determine IP address for RP
   - [recommended] Conect an extrenal monitor + keyboard to your RP to run the ... ifconfig ...  command to view the IP address assigned to the RP.
   - [alternative] Log into your router to identify the IP address of the RP (likely the most recently added device).
8. Prepare to set static IP address for RP
   - You need to set a static IP address for your RP so that 1) you know the IP address for remote access through SSH, and 2) services such as pi-hole don't freak out. Note: This is not the same as a static IP from your ISP. 
   - You can reserve your IP address for the RP in your router software. I won't go into detail here as router software varies significantly
     - [recommended] reserve an easy to remember IP address such as 10.0.0.10 (if you are using the 10.0.0.X series of addresses) or 192.168.0.192 (if you are using the 192.168.0.X series of addresses). Make it something you will remember because you will use it a lot.
     - [otherwise] use the automatically assigned IP for your RP, but change to static so that it does not change
   - You can probably perform the next step using 10) SSH but I expect you will lose connection after modifying the settings especially if the IP address is changed. You would then need to reconnect using SSH and the new IP. I have not tested this method and used my [recommended] monitor + keyboard connected to the RP.
   - [recommended] connect a monitor and keyboard to the pi. Proceed to the next step.
   - [not recommended] SSH into RPI Using your tool of choice (instructions are shown below. Proceed to the next step.
   - [untest] I recently discovered that you may be able to change these settings from the OMV menu after it has been installed. The dhcpcd.conf file is moved/modified by the OMV install.
9. Modify RP IP to static using the following commands: [from instructables: Configuring the Raspberry Pi Ethernet Port With a Static IP Address - author: ZRob314]
   - To view network status:
...
ifconfig
...
   - Backup network settings 
...    
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

TO CONFIGURE SSH:
- First update and note IP address using above procedure
- I use PuTTY for SSH (download here: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
- I don't use the hostname for SSH. I use the IP address of my RP. [see INSTALL RASPBERRY PI OS for notes on reserving a fixed IP for your RP]
- Log in using your RP info [see INSTALL RASPBERRY PI OS for setting username and password during setup]
- After updating some settings or after rebooting, you weill need to reconnect using the same process.

## PREPARE EXTERNAL HARDDRIVE

REFS:
- https://forums.raspberrypi.com/viewtopic.php?t=261455
- https://tldp.org/HOWTO/Partition/fdisk_partitioning.html

SUMMARY:
Partitioning and configuration of external harddrive for use with RP before installation of applications and containers. 

DETAILS:
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
- The main storage is recommended to be EXT4, however, I am using NTFS because I want to be able to plug my SSD into my windows computer to extract data if the RP crashes. There appear to be 3rd party tools that allow for a windows computer to read EXT4 (it is not read natively). I did not explore this option. If your main computer is Linux or Mac, EXT4 is probably a better option.
- The first three partitions are primary partitions. The NTFS partition is an extended partition (which allows for an extra layer of partitioning). Therefore, one extra sub-partition is required to "fill" this one.
- I only explicitely mount the SWAP drive on startup. I am not (yet) using the dedicated boot and root drives on the external harddrive. 

STEPS:
1. Connect the external drive to RP. A reboot may be required [unverified].
2. SSH into RP Using your tool of choice. 
3. Verify current drives and detected devices
   - usbls
   - df
   - bklid
4. Create partitions using fdisk
   -
5. Set-up SWAP drive and configuration
   -
   -


## INSTALL OPENMEDIAVAULT

REFS:
-

SUMMARY:

DETAILS:

STEPS:


To access OMV, type the IP address of the RP in the browser on a separate computer:
- http://10.0.0.10

## CONFIGURE EXTERNAL DRIVE IN OPENMEDIAVAULT

REFS:
-

SUMMARY:
The external harddrive must be mounted in OMV before use. I created shared folders on my external harddrive for all of my applications and containers. 

DETAILS:

STEPS:



Open OMV (type IP of RP in the browser).

Mount the NTFS partition
1. Storage> Disk> File Systems> (+)> Mount > 
  - [SELECT DRIVE PARTITION. ex: dev/sda3 NTFS XYZ GB]

To create new shared folders
1. Storage> Shared Folders> (+)> 
  - [ADD NAME - I use application name unless I have a reason to use something else]
  - [SELECT FILE SYSTEM - from previous step]
  - [RELATIVE PATH - for all apps, I created folders in var/lib/${APP NAME} to keep everything isolated]

## INSTALL DOCKER (OMV)

REFS:
-

SUMMARY:
Install DOCKER over OPENMEDIAVAULT

DETAILS:

STEPS:
1. Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
   - "docker": var/lib/docker/

## INSTALL PORTAINER (OMV)

REFS:
-

SUMMARY:
Install PORTAINER (GUI DOCKER MANAGER) over OPENMEDIAVAULT

DETAILS:

STEPS:
1. Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
   - "portainer": var/lib/portainer/


To access PORTAINER, type the IP address of the RP + port for the service in the browser on a separate computer:
- http://10.0.0.10:9000

## INSTALL HEIMDALL (PORTAINER)

REFS:
- https://hub.docker.com/r/linuxserver/heimdall


SUMMARY:

DETAILS:

STEPS:
1. Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
   - "heimdall": var/lib/heimdall/


To access HEIMDALL, type the IP address of the RP + port for the service in the browser on a separate computer:
- http://10.0.0.10:6941

TO ADD NEW SERVICE TO HEIMDALL:
1. Go to HIEMDALL service
2. Select (..) button to view services as list
3. Click add service, populating all fields (including url with http://10.0.0.10:PORT)
4. Click Save
5. Select (<- ->) button to rearrange buttons on main page

Add OPENMEDIAVAULT and PORTAINER to HEIMDALL [see steps for adding new service to HEIMDALL in INSTALL HEIMDALL (PORTAINER)]

## INSTALL SPEEDTEST TRACKER (PORTAINER)

REFS:
- https://github.com/henrywhitaker3/Speedtest-Tracker
- https://www.youtube.com/watch?v=nMNlkDPohcc

SUMMARY:
This is a small network speed test scheduling and record keeping tool that does not use a database (json instead).

STEPS:
1. Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
   - "speedtesttracker": var/lib/speedtesttracker/
2. Set-up DOCKER-COMPOSE for this container
   - Open PORTAINER [see how to access PORTAINER in INSTALL PORTAINER (OMV)]
   - Copy code from speedtesttracker.yml to editor
   - Copy absolute path of var/lib/speedtesttracker/ (should be something like: ) to the .yml script. 
     - Original:
     - Modified:
   - Deploy the docker-compose file.   




To access SPEEDTEST TRACKER, type the IP address of the RP + port for the service in the browser on a separate computer:
- http://10.0.0.10:8080

Add SPEEDTEST TRACKER to HEIMDALL [see steps for adding new service to HEIMDALL in INSTALL HEIMDALL (PORTAINER)]

## INSTALL PI-HOLE WITH UNBOUND (PORTAINER)

REFS:
- https://burakkarakan.com/blog/pihole-on-raspberry-using-pi-docker-and-docker-compose/
- https://github.com/pi-hole/docker-pi-hole
- https://docs.pi-hole.net/
- https://homenetworkguy.com/how-to/install-pihole-on-raspberry-pi-with-docker-and-portainer/
- https://github.com/willy-wagtail/raspberrypi#piholeandunboundwithdockercompose
- https://github.com/anudeepND/whitelist
- https://www.xfelix.com/2020/09/pihole-unbound-docker-setup-on-raspberry-pi/
- https://www.youtube.com/watch?v=4X6KYN1cQ1Y
- https://discourse.pi-hole.net/t/commonly-whitelisted-domains/212

SUMMARY:
Install PI-HOLE DNS 
DETAILS:

STEPS:
1. Set up new shared folders [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
   - var/lib//
   - var/lib//

To get dig function working
sudo apt install dnsutils

lists
https://firebog.net/

To access PI-HOLE, type the IP address of the RP + port for the service in the browser on a separate computer:
- http://10.0.0.4:8001/admin/:

Add PI-HOLE to HEIMDALL [see steps for adding new service to HEIMDALL in INSTALL HEIMDALL (PORTAINER)]

## INSTALL UNBOUND (PORTAINER)

REFS:
-

SUMMARY:

DETAILS:

STEPS:
1. Set up new shared folders [see CONFIGURE SSD IN OPENMEDIAVAULT]: var/lib//



## INSTALL NEXTCLOUD (PORTAINER) -- NOT CURRENTLY WORKING - ISSUES WITH OTHER USERS HAVING PERMISSION - ERROR 0770 -- 

REFS:
-

SUMMARY:

DETAILS:

STEPS:


Set up a new shared folder [see CONFIGURE SSD IN OPENMEDIAVAULT]: var/lib//

Add NEXCLOUD to HEIMDALL [see steps for adding new service to HEIMDALL in INSTALL HEIMDALL (PORTAINER)]

## CONFIGURE SMB FILE SHARE (OMV)
REFS:
- https://www.reddit.com/r/DataHoarder/comments/3xt15i/whats_the_name_of_your_nas/
- https://dannyda.com/2019/07/17/how-to-create-smb-cifs-windows-share-in-open-media-vault-omv/

SUMMARY:
Add ability for local folders to be shared on my network. In this case, I am creating one public folder accessible to everyone on the network (including guests), and one private folder only accessible to me (and select users with log on credentials).

DETAILS:
One of the most important steps when configuring a SMB file share/NAS is coming up with a good name [see the REDDIT link above]. I wanted the shared folders to be accessible from Windows computers and Android phones on my Home network. On a Windows computer, these folders show up under Networks in File Explorer. You will need to provide authentication credentials the first time you connect to the private drive. On Android, you will need an external app such as CX File Explorer to view the SMB file share. I needed the authentication credentials to view both the public and private folder [cause unknown].

STEPS:
1. Set up new shared folders [see CONFIGURE SSD IN OPENMEDIAVAULT]: 
- "NASa": shared/NASa public/
- "NASb": shared/NASb private/
2. Activate SMB file sharing on OMV
   -
4. Configure "NASa" and "NASb" as SMB file shares
   -
