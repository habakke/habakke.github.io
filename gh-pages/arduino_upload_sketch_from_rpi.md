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
sudo apt install arduino-mk
~~~

# Uploading sketch to Arduino

To upload a sketch to an Arduino connected using an USB cable from the Linux command line we first need to compile the Arduino sketch. The sketch project should already have a Makefile, so to build simply run make from the folder where the project is stored. This will create a folder build-<arduino-tag> which contains the compiled Arduino program.

~~~bash
make
~~~

To upload the compiled Arduino program, run make upload:

~~~bash
make upload
~~~

# View serial output from Arduino

To view the serial output form the Arduino run make monitor (requires screen)

~~~bash
make monitor
~~~

To exit the Arduino monitor press Ctrl+A and Ctrl+D. To reconnect to the screen run:

~~~bash
screen -r
~~~



