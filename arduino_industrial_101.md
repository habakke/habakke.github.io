Arduino Industrial 101 Setup
============================


# Configuration
This chapter details various aspects of setting up an arduino industrial 101 as a IOT node.

## Wifi configuration

If no wifi has been configured on the board previously, the arduino will create it's own wifi access point. 

If a wifi network has been configured on the board previously and the board is unable to connect to this network, press and hold the 
USER1 button (See diagram below) for 5-10 seconds to reset the wifi settings. The arduino will again create it's own wifi access point.

![Arduino Industrial 101](https://store.open-electronics.org/image/cache/catalog/data/Sistemidisviluppo-Softwareedidattica-Libri-Documentazionetecnica/7300-ARDUINO101/7300-ARDUINO101_4-500x500.jpg "Arduino Industrial 101")

The arduino access point will have the name arduino-yun-XXXXXXXX, where the X'es will be replaced with the mac addess of the wifi card 
on the device. Connect the computer to this network, point a browser to http://arduino.local. This will present the arduino configuration 
UI where the wifi network can be configured. The configuration UI will request a password, and the default password is "arduino".

## ZeroTier One configuration

In order for the device to become a proper IOT device, it should be able to connect to it's home server automatically. One way of 
acomplishing this is to install a ZeroTier One client on the device. The device will then setup a VPN connection to it's ZeroTier
controller. This can then be used to easily connect to the device, even though the actual IP of the device is unknown.

The ZeroTier client releases for OpenWRT can be found at https://github.com/mwarning/zerotier-openwrt/releases. The 
arduino industrial 101 require a package which is build for the ar71xx archictecture. Connect to the arduino using SSH and run 
the commands to install the client.

```bash
cd /tmp
wget https://github.com/mwarning/zerotier-openwrt/releases/download/v1.1.4_2/zerotier_1.1.4-2_ar71xx.ipk
opkg install zerotier_1.1.4-2_ar71xx.ipk
mkdir -p /var/lib/zerotier-one/networks.d
#zerotier-idtool generate /var/lib/zerotier-one/identity.secret /var/lib/zerotier-one/identity.public
/etc/init.d/zerotier start
```

Joining the Arduino Industrial 101 nodes network can be accomplished using the command below, where 'd5e5fb65375524b0' is the identifier of the Arduino Industrial 101 network.

```bash
zerotier-cli join d5e5fb65375524b0
```





