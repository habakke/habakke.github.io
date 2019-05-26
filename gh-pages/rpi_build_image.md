Building a RPI image
====================

The purpose of this document is to describe the procedure used to build a Raspberry PI software image. Once the image has been built and the wanted services deployed, a backup of the image should be created to simplify deploying the software image to other Raspberry PI's.

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
