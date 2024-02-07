# uKasa
A utility that manages Kasa smart outlets and switches using Asuswrt-Merlin routers

## Installation
uKasa can be installed on Asuswrt-Merlin routers.

Using ssh/shell, execute the following line:

For Asuswrt-Merlin based routers:

/usr/sbin/curl --retry 3 "https://raw.githubusercontent.com/JGrana01/uKasa/master/ukasa" -o "/jffs/scripts/ukasa" && chmod 0755 /jffs/scripts/ukasa && /jffs/scripts/ukasa install

__NOTE__ _ukasa_ requires Entware and bash. If bash is not installed, an error will be reported immediatly (not found).
To install bash on Asuswrt-merlin routers (assuming Entware has been installed via AMTM, perform these commands:

```
$ opkg update
$ opkg install bash
```



## About
uKasa is a small script/utility that lets you turn Kasa Smart Plugs and some Kasa Smart Switches off and on via the Linux CLI (or called from a script).
It also will show lots of information about the Kasa devices it finds on the local network including power consumption information if the plugs supports energy management.

uKasa attempts to support the Asuswrt-Merlin "AddOn" philosophy. It has an install and uninstall function and puts the executable script in /jffs/scripts (with a
symbolic link to /opt/bin) and install a "conf" file in /jffs/addons/ukasa.

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
```
 usage: ukasa <IP or hostname> <command>

   command can be...
       <on|off>         :  turn device off or on
       <state>          :  display the present state
       <power>          :  display the power (Voltage and  Watts) being used
       <monitor>        :  continually display the power (Voltage and  Watts) being used
       <info>           :  show information about a Kasa device

discover                :  search the network for Kasa devices
refresh                 :  re-create the list of all the devices discovered
devices                 :  show all the devices discovered
help (this screen)      :  show this screen
install                 :  run the install script
uninstall               :  remove ukasa files and directories
version                 :  show script version
update                  :  check for and update ukasa

```
ukasa can accept either the IP address of the Kasa device or the Linux hostname.

When installed, ukasa will scan the local network for devices. If the device is in the table, ukasa can perform commands on it.
If you add a new Kasa device or you believe the initil scan didn't find them all, perform the "_discovery_" command.

This command will scan the local network and add any new devices to the known device list. To see the device list and information about the Kasa device, issue the "_ukasa devices_" command.

Here is an example of the output from the command:

```Here are the Kasa devices found:
Device IP      Hostname          Model      Type    Features  Alias
---------------------------------------------------------------------------------
192.168.1.105  CasitaDeckLights  HS200(US)  Switch  TIM       Casita     Deck
192.168.1.106  DeckLights        HS200(US)  Switch  TIM       Porch      Lights
192.168.1.55   KP115US099914     KP115(US)  Plug    TIM:ENE   Test1
192.168.1.90   NetworkSys        KP125(US)  Plug    TIM:ENE   Christmas  Tree
192.168.1.92   WiFIWAN           KP125(US)  Plug    TIM:ENE   WiFi-WAN
192.168.1.93   TMOGateway        KP125(US)  Plug    TIM:ENE   TMO        Gateway
192.168.1.91   NetworkCloset     KP125(US)  Plug    TIM:ENE   Network    Closet
```
If a hostname can't be found, ukasa will create one with the model name and last 3 octects of the devices MAC address. Note Device IP 192.168.1.55 - ukasa created hostname.

## Commands

These command require the Device IP or Hostname of the Kasa device. I.e.  ukasa <_hostname or IP_>  <_command_>

_on/off_  - "on" and "off" will turn the device on or off

_state_  - display the present "on" or "off" state of the device
```
192.168.1.91    "Network Closet"        ON
```

_info_  - show a various information about the device
```
IP            Hostname       Model      Type  Features  Alias
192.168.1.91  NetworkCloset  KP125(US)  Plug  TIM:ENE   Network  Closet

Sofware Ver: 1.0.13   Hardwar Ver: 1.0  Model: KP125(US) ("Smart Wi-Fi Plug Mini")
  Device ID: 8006F3AD0AF3B86B87EE251210E096AE123456
WiFi rssi: -9   Power state: ON  MAC address: "1C:61:B4:11:22:33"  On time: 21d:6h:35m
Voltage: 122.2 (V)  Current: 989.0 (mA)  Power: 109.8 (W)  Total Watt Hours: 56659 (Wh)
```
_power_  - display the present voltage, current, power and Watt Hours of the device
```
Power Information Plug 192.168.1.91

Voltage: 122.4 (V)  Current: 910.0 (mA)  Power: 109.0 (W)  Total Watt Hours: 56664 (Wh)
```
__NOTE__ ukasa will report if the device does not support Energy Management (look for ENE in the Features column).

_monitor_  - continuos display (every 5 seconds) of the power being supplied by the device. Press the Enter key to stop.
```
Voltage: 121.0 (V)  Current: 917.0 (mA)  Power: 109.7 (W)  Total Watt Hours: 56699 (Wh)
```
These commands do not need a device identifier. They are informational or maintenance.

_discover_ - perform a network scan for Kasa devices. If one or more are found, add to the device list
__Note__ this command can take a few minutes for the scan and information retreival from the found device. Also, ignore the RTTVAR warnings, it's nmap making adjustments...
If a new device is discovered, it will be added. Devices are NOT removed from the list. To do a compete rescan of the local network, use the command _refresh_.

_refresh_ - This command is identical to _discover_ except it completely re-creates the internal devices list/file. If you remove plugs or switches, you will need to do a "refresh".

_devices_ - this command will display a table of the presently known Kasa devices it has discoveded
```
Here are the Kasa devices found:

Device IP      Hostname          Model      Type    Features  Alias
---------------------------------------------------------------------------------
192.168.1.105  CasitaDeckLights  HS200(US)  Switch  TIM       Casita     Deck
192.168.1.106  DeckLights        HS200(US)  Switch  TIM       Porch      Lights
192.168.1.55   KP115US099914     KP115(US)  Plug    TIM:ENE   Test1
192.168.1.90   NetworkSys        KP125(US)  Plug    TIM:ENE   Christmas  Tree
192.168.1.92   WiFIWAN           KP125(US)  Plug    TIM:ENE   WiFi-WAN
192.168.1.93   TMOGateway        KP125(US)  Plug    TIM:ENE   TMO        Gateway
192.168.1.91   NetworkCloset     KP125(US)  Plug    TIM:ENE   Network    Closet
```

_install_ - Install ukasa. Install first checks for apps that it requires - nmap, jq, od and base64. If any are not installed, ukasa installs them via opkg. The install script will then create a directory in /jffs/addons/ukasa and create a small config file (/jffs/addons/ukasa/ukasa.conf).
It then performs and nmap scan of the local network to discover Kasa plugs and switches on the network. For every one found, ukasa attempts to find the local hostname of the device. It first looks in /etc/hosts then YazDHCP if it is installed.
If it can't determine the hostname, it creates one by using the Type of device and the last 3 octects of it's MAC address (i.e. KP125AF0155). ukasa then adds an entry to the devices file  /jffs/addons/ukasa/ukasa.devices with the devices IP address, it's local hostname, model, type of device, features and alias (name of the device assigned by the Kasa iOS or Android app.
Install will then offer to install a directory in /jffs/addons/ukasa with one or more example scripts that use ukasa.

_uninstall_ - Uninstall will remove all traces of ukasa - the script itself, the link in /opt/sbin and it's directory in /jffs/addons.

_update_ - update checks for the latest version of ukasa on github and if newer will offer to download it over the existing script. It will also optionally re-install the examples.

_version_ - prints the version of the installed ukasa

_help_ - prints a help screen showing commands and usage

## Notes

At this time, ukasa __does not suport_ the Matter enabled plugs and switches. These models typically end the model number in an "M"

The _discover_ command sometimes does not get a response from a plug or switch. Typically running it a few times finds them all







_







