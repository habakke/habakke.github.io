RPI IoT node Configuration 
==========================

The purpose of this document is to describe how to build a IoT Pinode software image which is used by a Raspberry PI to serve as an IoT controller node.

Setting up the services are detailed in separate documents, using the IoT Pinode as a base platform.

# Requirements

* Raspberry PI 3+
* 16Gb SD card
* USB Keyboard
* Screen with HDMI input
* Wired or wireless network connection
* Micro USB power adapter

# Software Image Configuration

The IoT Pinode software image builds upon the standard Rasbian software image for Raspberry PI. To do a headless Raspberry PI install start by downloading the latest Rasbian Stretch lite software image from [Rasbian](https://downloads.raspberrypi.org/raspbian_lite_latest) and extract the downloaded .zip file.

Write the extracted .img file to a SD card using [Etcher](https://etcher.io). Once Etcher has successfully written the image to the SD card reconnect the SD card.

## Enable SSH

To enable SSH by default when booting, create a file called ssh.txt in the root folder on the SD card.

## Enable HDMI output

To enable HDMI output, edit the config.txt file located in the root of the SD card. Add the following line to the file:

~~~bash
hdmi_force_hotplug=1
~~~ 

## Setting up wireless networking

To setup the wireless networking automatically upon first boot, create a file called wpa_supplicant.conf for your particular wireless network in the root folder on the SD card. The file must contain the following lines:

~~~bash
country=no
update_config=1
ctrl_interface=/var/run/wpa_supplicant

network={
    scan_ssid=1
    ssid="SSID_1"
    psk="XXXXXX"
    key_mgmt=WPA-PSK
    priority=1
    id_str="KRS"
}

network={
    scan_ssid=1
    ssid="SSID_2"
    psk="XXXXXXXX"
    key_mgmt=WPA-PSK
    priority=2
    id_str="OSL"
}
~~~ 

To change the wireless network configuration once you have successfully booted and connected to the Raspberry PI, use the included raspberry pi configuration tool. From the menu select (2) Network Options and (N2) Wi-fi. Then enter the correct SSID and password for the selected network.

~~~bash
sudo raspi-config
~~~ 

## First boot

After first boot, upgrade the installed packages and restart the system.

~~~bash
sudo apt-get update && sudo apt-get upgrade -y
sudo reboot
~~~ 

## SSH public/private key auth configuration

To improve security and ease use, setup public/private key authentication. If you dont have a key already, start by generating a key locally.

~~~bash
ssh-keygen
~~~ 

Assuming the default SSH key name was used in the above step (the key will be named "id_rsa"), copy the key to the RPI.

~~~bash
ssh-copy-id -i ~/.ssh/id_rsa pi@<rpi-ip>
~~~ 

## Pinode customization

Login to the raspberry pi (The default user is  pi, and the password is raspberry.) and start executing the commands below.

### Disable WiFi and Bluetooth

To completely disable WiFi and Bluetooth, effectively putting the RPI in airplane mode, edit the file '/etc/modprobe.d/raspi-blacklist.conf'

To disable WiFi, disable loading of the Broadcom WiFi drivers by adding these lines to the file:

~~~bash
blacklist brcmfmac
blacklist brcmutil
~~~

To disable Bluebooth, disable loading of the Broadcom Bluetooth drivers by adding these lines to the file:

~~~bash
blacklist btbcm
blacklist hci_uart
~~~
 
### Configuring USB automounting

To configure support for USB automounting, start by adding support for NTFS file system.

~~~bash
sudo apt-get install ntfs-3g
~~~

Format the USB device with a single NTFS partition. Verify that inserting the USB device into the RPI, the device shows up as /dev/sda1.

~~~bash
ls -l /dev/sd*
~~~

Edit the /etc/rc.local file, adding the command to  start the usbmount script just above the last line in the file. The file should then look like this

~~~
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi

/opt/python-library/usbmount.sh & > /tmp/usbmount.log 2>&1

exit 0
~~~

### Setup ZeroTier One Service

In order for the device to become a proper IOT device, it should be able to connect to it's home server automatically. One way of acomplishing this is to install a ZeroTier One client on the device. The device will then setup a VPN connection to it's ZeroTier controller. This can then be used to easily connect to the device, even though the actual IP of the device is unknown.

~~~bash
curl -s https://install.zerotier.com/ | sudo bash
~~~ 

Joining the RPI Recorder nodes network can be accomplished using the command below, where 'abfd31bd47683754' is the identifier of the Arduino Industrial 101 network.

~~~bash
sudo zerotier-cli join abfd31bd47683754
~~~ 

## Building Pinode image

Once the image has been built and the wanted services deployed, a backup of the image should be created to simplify deploying the software image to other Raspberrry pi's.

### Mac OSX

Connect the SD card and use the command below to list the devices connected and find disk name for the SD card:

~~~bash
diskutil list

/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *240.1 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                 Apple_APFS Container disk1         239.8 GB   disk0s2

/dev/disk1 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +239.8 GB   disk1
                                 Physical Store disk0s2
   1:                APFS Volume SSD                     46.9 GB    disk1s1
   2:                APFS Volume Preboot                 43.2 MB    disk1s2
   3:                APFS Volume Recovery                514.0 MB   disk1s3
   4:                APFS Volume VM                      3.2 GB     disk1s4

/dev/disk2 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *15.9 GB    disk2
   1:             Windows_FAT_32 boot                    46.0 MB    disk2s1
   2:                      Linux                         15.9 GB    disk2s2
~~~ 

As seen from the output above, the SD card has the name /dev/disk2. To create an image from this SD card, run the following command:

~~~bash
sudo dd if=/dev/disk2 of=./pinode.img
~~~ 

Writing the image back to the SD card can be done by unmounting the SD card and then using the reverse command:

~~~bash
diskutil unmountDisk /dev/disk2
sudo dd if=~/pinode.img of=/dev/disk2
~~~ 

Once it has finished writing the image to the SD card, you can remove it from your Mac using:

~~~bash
sudo diskutil eject /dev/disk2
~~~ 

### Linux

Connect the SD card and use the command below to list the devices connected and find disk name for the SD card:

~~~bash
df -h

Filesystem 1K-blocks Used Available Use% Mounted on
rootfs 29834204 15679020 12892692 55% /
/dev/root 29834204 15679020 12892692 55% /
devtmpfs 437856 0 437856 0% /dev
tmpfs 88432 284 88148 1% /run
tmpfs 5120 0 5120 0% /run/lock
tmpfs 176860 0 176860 0% /run/shm
/dev/mmcblk0p1 57288 14752 42536 26% /boot
/dev/sda5 57288 9920 47368 18% /media/boot
/dev/sda6 6420000 2549088 3526652 42% /media/41cd5baa-7a62-4706-b8e8-02c43ccee8d9
~~~ 

As seen from the output above, the SD card has the name /dev/mmcblk0p1. The last part ('p1' or '1') is the partition number, but you want to use the whole SD card, so you need to remove that part from the name leaving '/dev/mmcblk0' as the disk you want to read from. To create an image from this SD card, run the following command:

~~~bash
sudo dd if=/dev/mmcblk0 of=./pinode.img
~~~ 

Writing the image back to the SD card can be done by unmounting the SD card and then using the reverse command:

~~~bash
sudo umount /dev/mmcblk0
sudo dd bs=4M if=~/pinode.img of=/dev/mmcblk0
~~~ 

Wait while it completes. Before ejecting the SD card, make sure that your Linux PC has completed writing to it using the command:

~~~bash
sudo sync
~~~ 

### Windows

TBD.

The end.
