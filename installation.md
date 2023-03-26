# Installation Guide

## Hardware Requirements

For the Secure Raspberry Pi Router, I used following hardware components:

- Raspberry Pi 4 Model B Rev 1.4, 4 GB, 1.5Hz
- RT5370 150 Mbps Wireless USB Dongle Stick Adapter
- Samsung Evo+ microSDHC, 32 GB

## Software Requirements


- OpenWrt (latest stable version recommended). Download from the [official website](https://openwrt.org/)
- A VPN service, such as ProtonVPN or NordVPN.

## Installation Steps

### Install OpenWRT
Download [OpenWRT OS](https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi) for your Raspberry model from the homepage and use the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) to install it to the SD card.

1. Operating System -> Use Custom and select the OpenWRT image.
2. Select your SD card as storage
3. Click WRITE

Boot the Pi with the SD card and connect your computer directly to the PI using an ethernet cable. Your computer needs a fixed IP address such as 192.168.1.10 for this. OpenWRT has by default the address 192.168.1.1

Connect to the PI with SSH:
```
ssh root@192.168.1.1
```

By default you connect to OpenWRT with the root user without password. Optionally you can set a password for the root user with `passwd`.

### Configure Network
Open network configuration: `vi /etc/config`

Add static private ip address to the lan interface e.g. 10.71.71.1
```
config interface 'lan'
        option device 'br-lan'
        option proto 'static'
        option ipaddr '10.71.71.1'
        option netmask '255.255.255.0'
        option ip6assign '60'
        option force_link '1'
```

Create an interface for the Wifi that is later used to connect the clients to the network with DHCP. Add some DNS server such as Cloudflare and Google to the config.
```
config interface 'wwan'
        option proto 'dhcp'
        option peerdns '0'
        option dns '1.1.1.1 8.8.8.8'
```

Add a interface for the VPN client:
```
config interface 'vpnclient'
        option ifname 'tun0'
        option proto 'none'
```

### Configure Firewall
Open firewall configuration: `vi /etc/firewall`

Allow input and output traffic on wan network:
```
config zone                                
        option name             wan        
        list   network          'wan'      
        list   network          'wan6'          
        option input            ACCEPT          
        option output           ACCEPT          
        option forward          REJECT          
        option masq             1               
        option mtu_fix          1  
```

### Connect to OpenWRT via DHCP
After network and firewall is configured, reboot the PI. Connect again via SSH with the IP address you just set: `ssh root@10.71.71.1`

OpenWRT is now configured as DHCP and should hand out IP addresses. Therefore your computer settings should be set to DHCP. Your computer should get an IP in the range of 10.71.71.1 - 10.71.71.254.

### Configure wifi device
Open wireless configuration: `vi /etc/wireless`

Configuration radio interface:
```
config wifi-device 'radio0'
        option type 'mac80211'
        option path 'platform/soc/fe300000.mmcnr/mmc_host/mmc1/mmc1:0001/mmc1:0001:1'
        option channel '7'
        option band '11g'
        option htmode 'HT20'
        option disabled '0'
        option short_gi_40 '0'
```

Enable wifi device:
```console
root@OpenWrt:~# uci commit wireless
root@OpenWrt:~# wifi
```

If you scan the WIFI around you, some network with name Open WRT should show up.

### Connect OpenWRT to Wireless Network
1. Open Web GUI at: http://10.71.71.1
2. Go to Network -> Wireless
3. Click the Scan button and join the your Wifi network
4. Enter the passphrase and make sure to click the `Replace wireless configuration` checkbox

### Setup RT5370 Wireless Adapter Access Point
Install packages:
```sh
opkg update
opkg install kmod-rt2800-lib kmod-rt2800-usb kmod-rt2x00-lib kmod-rt2x00-usb kmod-usb-core kmod-usb-uhci kmod-usb-ohci kmod-usb2
```

Plugin the adapter to the Raspberry Pi USB2.0 port and list all usb devices
```console
root@OpenWrt:~# opkg update
root@OpenWrt:~# opkg install usbutils
root@OpenWrt:~# lsusb
Bus 001 Device 003: ID 148f:7601  802.11 n WLAN
Bus 002 Device 001: ID 1d6b:0003 Linux 5.10.161 xhci-hcd xHCI Host Controller
Bus 001 Device 002: ID 2109:3431  USB2.0 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux 5.10.161 xhci-hcd xHCI Host Controller
```
It should be listed as 802.11 n WLAN

Enable WLAN adapter with: `ifconfig wlan1 up`

### Setup VPN Connection
Install packages:
```sh
opkg update
opkg install openvpn-openssl luci-app-openvpn
```

Go to to Web UI: http://10.71.71.1 VPN -> OpenVPN

Follow this instructions for [ProtonVPN](https://protonvpn.com/support/how-to-set-up-protonvpn-on-openwrt-routers/)