RPInode Configuration 
=====================

The purpose of this document is to describe how to build a Pinode software image which is used by a Raspberry PI to host one or more python based services.

Setting up the services are detailed in separate documents, using the Pinode as a base platform.

# Requirements

* Raspberry PI 3+
* 16Gb SD card
* USB Keyboard
* Screen with HDMI input
* Wired or wireless network connection
* Micro USB power adapter

# Software Image Configuration

The Pinode software image builds upon the standard Rasbian software image for Raspberry PI. To do a headless Raspberry PI install start by downloading Raspberry Pi Imager from [Rasbian](https://www.raspberrypi.org/downloads/) and install the application. Use the application to install Rasbian Lite onto the SD card. Once Raspberry Pi Imager has successfully written the image to the SD card reconnect the SD card.

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

### Install Pip

Install the python package manager

~~~bash
sudo apt-get install -y python3-pip
~~~ 

### Optional: Set python3/pip3 as default python environment

To configure the raspberry pi to use the python3 configure python alternatives. First list the available python versions

~~~bash
ls /usr/bin/python*

/usr/bin/python          /usr/bin/python2.7-config  /usr/bin/python3.5m
/usr/bin/python-config   /usr/bin/python3           /usr/bin/python3.5m-config
/usr/bin/python2         /usr/bin/python3-config    /usr/bin/python3m
/usr/bin/python2-config  /usr/bin/python3.5         /usr/bin/python3m-config
/usr/bin/python2.7       /usr/bin/python3.5-config
~~~ 

Then create a configuration alternative for each python version

~~~bash
update-alternatives --install /usr/bin/python python /usr/bin/python2 1
update-alternatives --install /usr/bin/python python /usr/bin/python3 2
~~~ 

Then create a configuration alternative for each pip version

~~~bash
update-alternatives --install /usr/bin/pip2 pip /usr/bin/pip2 1
update-alternatives --install /usr/bin/pip3 pip /usr/bin/pip3 2
~~~ 

From now on, we can anytime switch between the above listed python alternative versions using below command and entering a selection number:

~~~bash
update-alternatives --config python

There are 2 choices for the alternative python (providing /usr/bin/python).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python3.5   2         auto mode
  1            /usr/bin/python2.7   1         manual mode
  2            /usr/bin/python3.5   2         manual mode
~~~ 

### Setup Python virtual environment

A python virtual environment is a special tool used to keep the dependencies required by different projects in separate places by creating isolated, independent Python environments for each of them. It is standard practice to use a virtual environment when using python so not to mess upp the system packages. 

Setup a python virtual environment:

~~~bash
sudo pip3 install virtualenv virtualenvwrapper
sudo rm -rf ~/.cache/pip
~~~ 

Now that both virtualenv and virtualenvwrapper have been installed, we need to update our ~/.profile file to include the following lines at the bottom of the file:

~~~bash
echo -e "\n# virtualenv and virtualenvwrapper" >> ~/.profile
echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.profile
echo "export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3" >> ~/.profile
echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.profile
~~~ 

Now that we have our ~/.profile updated, we need to reload it to make sure the changes take affect. You can force a reload of your ~/.profile file by:

~~~bash
source ~/.profile
~~~ 

Create a Python virtual environment that we’ll use for pinode:

~~~bash
mkvirtualenv pinode -p python3
~~~

The pinode Python virtual environment is entirely independent and sequestered from the default Python version included in the download of Raspbian. Any Python packages in the global site-packages directory will not be available to the pinode virtual environment. Similarly, any Python packages installed in site-packages of pinode will not be available to the global install of Python. Keep this in mind when you’re working in your Python virtual environment and it will help avoid a lot of confusion and headaches.

If you ever reboot your Raspberry Pi; log out and log back in; or open up a new terminal, you’ll need to use the workon command to re-access the pinode virtual environment. After that, you can use workon and you’ll be dropped down into your virtual environment:

~~~bash
source ~/.profile
workon pinode
~~~

To validate and ensure you are in the pinode virtual environment, examine your command line — if you see the text (pinode) preceding your prompt, then you are in the pinode virtual environment

# Setting up Pinode services

The following chapters describes how to setup the available services on the Pinode platform. One or more services may be installed simultaneously.
 
## Video Recording Service

The video recording service require the Raspberry PI to be equipped with a touch screen and an analog video capture card.

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

### Configuring Adafruit PiTFT 3.5 for use by the Video Recording Service

To setup and configure the Adafruit PiTFT touchscreen display start by executing the following commands:

~~~bash
cd ~
wget https://raw.githubusercontent.com/adafruit/Raspberry-Pi-Installer-Scripts/master/adafruit-pitft.sh
chmod +x adafruit-pitft.sh
sudo ./adafruit-pitft.sh
~~~ 

Once you run it you will be presented with menus for configuration. From the menu select option 4. PiTFT 3.5" resistive touch (320x480)

~~~bash
This script downloads and installs
PiTFT Support using userspace touch
controls and a DTO for display drawing.
one of several configuration files.
Run time of up to 5 minutes. Reboot required!

Select configuration:
1. PiTFT 2.4", 2.8" or 3.2" resistive (240x320)
2. PiTFT 2.2" no touch (240x320)
3. PiTFT 2.8" capacitive touch (240x320)
4. PiTFT 3.5" resistive touch (320x480)
5. Quit without installing

SELECT 1-5: 
~~~ 

Next you will be asked for the rotation you want, select option 1. 90 degrees (landscape)

~~~bash
Select rotation:
1. 90 degrees (landscape)
2. 180 degrees (portait)
3. 270 degrees (landscape)
4. 0 degrees (portait)

SELECT 1-4:
~~~ 

Next the script will ask two questions to determine how the display should be used. We want the display to be used exclusively as a raw framebuffer device. To enable this configuration answer NO to both of the questions:

~~~bash
Would you like the console to appear on the PiTFT display? [y/n] n
Would you like the HDMI display to mirror to the PiTFT display? [y/n] n
~~~ 

Finally the script will ask for permission to reboot the device. Select yes to allow the device to reboot.

~~~bash
[PITFT] Success!

Settings take effect on next boot.

REBOOT NOW? [y/N] 
~~~ 

Wait for the Raspberry pi to reboot. 

Now that the screen is working nicely, we'll take care of the touchscreen. There's just a bit of calibration to do, but it isn't hard at all.

The Rasbian OS should already be pre-configured to work with the PiTFT 3.5 touchscreen. Verify the touchscreen device is available:

~~~bash
ls -l /dev/input/touchscreen

lrwxrwxrwx 1 root root 6 Jan 16 21:46 /dev/input/touchscreen -> event0
~~~ 

There are some tools we can use to calibrate & debug the touchscreen. Install the "event test" and "touchscreen library" packages with

~~~bash
sudo apt-get install -y evtest tslib libts-bin
~~~ 

### Configuring PICapture HD1 for use by the Video Recording Service

The PICapture HD1 HAT for the raspberry pi is used for capturing analog and digital video. To setup the card, the Raspberry Pi Camera driver must be enabled. Start the raspi-config tool and navigate to "5 Interfacing Options -> P1 Camera" and enable the camera interface.

~~~bash
sudo raspi-config
~~~ 

The capture card also requires I2C for control. Make sure I2C is enabled using raspi-config again. Navigate to "5 Interfacing Options -> P5 I2C" and enable the I2C kernel module.

~~~bash
sudo raspi-config
~~~ 

For the changes above to take effect, we need to reboot.

~~~bash
sudo reboot
~~~ 

Finally install the PI capture control software. Since we are using I2C control interface, we need to install the python-smbus package in addition to the PyPi picapture package.

~~~bash
sudo apt-get install -y python3-serial python3-smbus
sudo pip3 install pivideo
~~~ 

To verify the PI capture card is installed correctly and the software is working, run the self test.

~~~bash
sudo pivideo -q all

PiCapture HD1 is ready
No active video detected
Selected video source is:  auto
Active video source is:  component
Raspberry Pi camera port is not active
Video processor firmware version:  05
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


### Setup Video Recording Service

Start the installation by downloading the latest software by cloning the git repostitory:

~~~bash
cd /opt
sudo mkdir python-library
sudo chown pi:pi python-library
cd python-library
sudo apt-get install -y git
git clone https://habakke@bitbucket.org/team-bluebit/python-library.git .
~~~ 

Install the python-library dependencies. Start by installing pygame which have to be installed through apt-get and not pip:

~~~bash
sudo apt-get install -y python3-pygame
~~~ 

The rest of the dependencies can be installed the normal way

~~~bash
cd /opt/python-library
sudo pip3 install -r requirements_rpi.txt
~~~ 

Fix any issues that occur during installation of the packages and list the installed python packages to verify the install:

~~~bash
sudo pip3 list
~~~ 

Install system libraries required by the service

~~~bash
sudo apt-get install -y libsdl1.2-dev libhdf5-100 libharfbuzz0b libwebp6 libtiff5 libjasper1 libilmbase12 libopenexr22 libgstreamer1.0-0 libavcodec-extra57 libavformat57 libswscale4 libgtk-3-0 libsdl-ttf2.0-dev

sudo apt-get install -y liblapack3
sudo apt-get install -y libcblas3
sudo apt-get install -y libatlas-base-dev
sudo apt-get install -y libqtgui4
sudo apt-get install -y libqt4-test
~~~ 

Give the necessary access rights to the 'pi' user to access /dev/tty and /dev/console

~~~bash
sudo addgroup --system console
sudo chgrp console /dev/console
sudo chmod g+rw /dev/console
sudo usermod -a -G console pi
sudo usermod -a -G tty pi
~~~ 

Start the video recording service manually to verify the setup

~~~bash
cd /opt/python-library
python src/fbtest.py --test -r 480x320
~~~ 

Configuring video recording service to run as a service by creating a recorder.service file in /etc/systemd/system folder:

~~~
[Unit]
Description=Video Recording service 
Wants=network-online.target
After=rsyslog.service
After=network-online.target

[Service]
Restart=yes
ExecStart=/opt/python-library/start.sh
ExecStop=/bin/kill -INT $MAINPID
OOMScoreAdjust=-100
TimeoutStopSec=10s
User=root
WorkingDirectory=/opt/python-library
StandardInput=tty
StandardOutput=tty
TTYPath=/dev/tty2

[Install]
WantedBy=multi-user.target
Alias=recorder
~~~

Issue these commands in the command prompt (I assume recorder.service file is already placed to the correct location where systemd can find it):

~~~bash
sudo systemctl daemon-reload
sudo systemctl start recorder.service
~~~

The end.
