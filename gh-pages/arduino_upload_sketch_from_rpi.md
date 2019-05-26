Upload Arduino sketch from Raspberry PI
=======================================

The purpose of this document is to describe how to install the necessary components to upload an Arduino sketch to an Arduino connected to a Raspberry PI using an USB cable.

# Requirements
* Raspberry PI 3
* Arduino (any model)
* USB cable
* Arduino sketch

# Installing Arduino software on Raspberry PI

Login to the Raspberry PI and start by downloading the latest Arduino Linux binaries and extract to '/opt/arduino'

~~~bash
cd /opt
sudo wget https://www.arduino.cc/download_handler.php?f=/arduino-1.8.9-linuxarm.tar.xz
sudo apt install xz-utils
sudo tar xf arduino-1.8.9-linuxarm.tar.xz
cd arduino
~~~

# Uploading sketch to Arduino

To upload a sketch to an Arduino connected using an USB cable, use the following command:

~~~bash
/opt/arduino/arduino --upload <path to sketch> --port /dev/ttyUSB*
~~~

# View serial output from Arduino

To view the serial output form the Arduino and verify the sketch was successfully uploaded, minicom can be used. (Insert the correct baud rate specified in the Serial.begin(<baud>) command)

~~~bash
minicom -D /dev/ttyUSB* -b <baud rate>
~~~
