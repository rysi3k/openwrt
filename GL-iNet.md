# Configuring a GL-iNet device for monitoring APC UPS with NRPE

## Installing most up-to-date version of the software

* Connect WAN port to internet
* Power on device
* Connect to LAN port, assign local machine IP in range 192.168.8.0/24 (not .1)
* Browse to http://192.168.8.1 and follow instructions to set password
* Go to Firmware link on left-hand side, upgrade to the newest version (2.27)
  * Untick 'Keep settings' to ensure a clean install
  * If no internet connection available, download firmware from
    <https://www.gl-inet.com/firmware/ar150/v1/> (as of May 2018,
    lede-ar150-2.27.bin) and upload
* Browse to 192.168.8.1 and set password
* Powercycle system (this enables SSH)
* Connect via SSH: `ssh -oUserKnownHostsFile=/dev/null root@192.168.8.1`

## Configure the device

Set hostname:

* Edit `/etc/config/system` and modify the `option hostname 'GL-AR150'` line
* `/etc/init.d/system restart`

Disable wireless:

* Edit `/etc/config/wireless` and to `config wifi-iface 'default_radio0'` stanza add:

      option disabled '1'


### Make services available over wan as well as lan interface

PoE only works over the WAN port.  To reduce the number of ports required to
make the device function, the WAN port can be configured similarly to the LAN
port.

* Edit `/etc/config/firewall` - to config zone named "lan", set:

      option network 'lan wan'

  and to config zone named "wan" set:

      option network 'wan6'

* `/etc/init.d/firewall` restart

### Disable functions on LAN port

If using the LAN port, you may well not want DHCP running.

To disable DHCP on LAN port:

* Edit `/etc/config/dhcp` and in the `config dhcp 'lan'` stanza set:

      option ignore '1'

  remove the following if present:

      option dhcpv6 'server'
      option ra 'server'

* `/etc/init.d/dnsmasq restart`
* `/etc/init.d/odhcpd restart`

## Install apcupsd

Ensure connected to internet, then:

* `opkg update`
* `opkg install kmod-usb-hid`
* `opkg install apcupsd`

* Edit `/etc/apcupsd/apcupsd.conf`:

      UPSNAME location_name
      UPSCABLE usb
      UPSTYPE usb

  Ensure that `DEVICE` is not defined (comment out)

* Restart: `/etc/init.d/apcupsd restart`
  (if you get this error:

      cat: can't open '/var/run/apcupsd.pid': No such file or directory
      sh: you need to specify whom to kill

  you need to `/etc/init.d/apcupsd restart` again)

* Test by running `apcaccess`

      root@GL-AR150-new:~# apcaccess
      APC      : 001,037,0930
      DATE     : 2018-06-10 19:02:47 +0100  
      HOSTNAME : GL-AR150-new
      VERSION  : 3.14.14 (31 May 2016) unknown
      UPSNAME  : ups1
      CABLE    : USB Cable
      DRIVER   : USB UPS Driver
      UPSMODE  : Stand Alone
      STARTTIME: 2018-06-10 17:08:48 +0100  
      MODEL    : Back-UPS RS 900G 
      STATUS   : ONBATT 
      LINEV    : 0.0 Volts
      LOADPCT  : 0.0 Percent
      BCHARGE  : 27.0 Percent
      TIMELEFT : 683.4 Minutes
      MBATTCHG : 5 Percent
      MINTIMEL : 3 Minutes
      MAXTIME  : 0 Seconds
      SENSE    : Medium
      LOTRANS  : 176.0 Volts
      HITRANS  : 294.0 Volts
      ALARMDEL : No alarm
      BATTV    : 25.7 Volts
      LASTXFER : No transfers since turnon
      NUMXFERS : 2
      XONBATT  : 2018-06-10 19:02:43 +0100  
      TONBATT  : 8 Seconds
      CUMONBATT: 5935 Seconds
      XOFFBATT : 2018-06-10 18:48:07 +0100  
      SELFTEST : OK
      STATFLAG : 0x05060010
      SERIALNO : 3B1733X16096  
      BATTDATE : 2017-08-19
      NOMINV   : 230 Volts
      NOMBATTV : 24.0 Volts
      NOMPOWER : 540 Watts
      FIRMWARE : 879.L4 .I USB FW:L4
      END APC  : 2018-06-10 19:02:51 +0100

(From <https://wiki.openwrt.org/doc/howto/apcupsd_es500>)

## Install monitoring packages

Follow README.md

## Install the checks

* Download check:
    
      wget -O /usr/sbin/check_apcupsd http://lancet.mit.edu/mwall/projects/nagios/check_apcupsd
      chmod +x /usr/sbin/check_apcupsd

* Edit `/usr/sbin/check_apcupsd` and modify `APCACCESS=/sbin/apcaccess` to `APCACCESS=/usr/sbin/apcaccess`

* Add lines to `/etc/nrpe.cfg`:

      command[check_apc_status]=/usr/sbin/check_apcupsd status
      command[check_apc_charge]=/usr/sbin/check_apcupsd charge
      # temp isn't implemented
      #command[check_apc_temp]=/usr/sbin/check_apcupsd temp
      command[check_apc_load]=/usr/sbin/check_apcupsd load
      command[check_apc_time]=/usr/sbin/check_apcupsd time


## Recovering if you brick the device

* Download firmware from <https://www.gl-inet.com/firmware/ar150/v1/> (as of May 2018, lede-ar150-2.27.bin)
* Remove all cables
* Add the LAN cable
* Hold down the Reset button
* Add power
* Wait for 5-6 flashes of the red LED, until the middle green LED turns on
* Release the reset button
* Device will now have IP 192.168.1.1
* Navigate to web interface, and upload new firmware
