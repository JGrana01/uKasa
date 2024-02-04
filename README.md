# uKasa
A utilty that manages Kasa smart outlets and switches in Asuswrt-Merlin routers or Linux RPi

## Installation
uKasa can be installed on Debian/Linux systems (i.e. Raspberry Pi) and Asuswrt-Merlin routers.

Using ssh/shell, execute the following line:

For Asuswrt-Merlin based routers:

/usr/sbin/curl --retry 3 "https://raw.githubusercontent.com/JGrana01/uKasa/master/ukasa" -o "/jffs/scripts/ukasa" && chmod 0755 /jffs/scripts/ukasa && /jffs/scripts/ukasa install

For Linux (i.e. Raspberry Pi)

/usr/bin/curl --retry 3 "https://raw.githubusercontent.com/JGrana01/uKasa/master/ukasa" -o "$HOME/ukasa" && chmod 0755 $HOME/ukasa && $HOME/ukasa install

## About
uKasa is a small script/utility that lets you turn Kasa Smart Plugs and some Kasa Smart Switches off and on via the Linux CLI (or called from a script).
It also will show lots of information about the Kasa devices it finds on the local network including power consumption information if the plugs supports energy management.

uKasa attempts to support the Asuswrt-Merlin "AddOn" philosophy. It has an install and uninstall function and puts the executable script in /jffs/scripts (with a
symbolic link to /opt/bin) and install a "conf" file in /jffs/addons/ukasa.

For Linux/Raspbian installations, the executable script will be installed in /usr/local/sbin and the conf file in the $HOME/.config/ukasa directory

During install, ukasa can download a small group of example scripts using using ukasa. If selected, these will be downloaded to the ~ukasa/examples directory.
The main reason I ported and worked on this script was to be able to power cycle a few of my AiMesh nodes. I had some problems with one node going offline.
So, I used ukasa to remotly power cycle the router.

## Installation Process

When ukasa installs, it checks/downloads apps it needs (jq, column, od and nc), sets up the configuration directory with a config file (ukasa.conf).
It then attempts to find any Kasa Smart Plugs and Switches on the local network. It does this using nmap and scans for any open ports at 9999.
If one is found, it then looks up it's IP address and hostname. The hostname can be one from /etc/hosts, YazDHCP or will be created by using the last 3 MAC address octets.
ukasa then probes the device and attempts to get the devices type (plug, switch or unknown), it's featurs and the name (aka alias) you gave the device when you set it up with the Kasa app.
ukasa stores this information in a local file.

Once completed, it displays all the plug/switches it found along with the information it recevied or generated.

__NOTE__ nmap is usually pretty good about finding all the Kasa devices. Once in a while it might miss one. If the number of devices it reports found doesn't match with what you believe, run ukasa in discover mode

__NOTE2__ at this time ukasa _doesn't_ support the new matter based plugs (i.e. KP125M).

## Usage
ukasa supports on and off commands along with power reporting and detailed information about the device.





