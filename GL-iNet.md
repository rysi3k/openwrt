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

* Test with `apcaccess`

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
