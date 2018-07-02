# Building NRPE for OpenWRT on the GL-AR150

In general, follow <http://nerdbynature.de/s9y/2016/10/03/Building-NRPE-for-OpenWRT>.

## Step-by-step guide

*   These instructions are based on a Debian stretch install (previous versions of OpenWRT required jessie for libssl compatibility reasons, but the May 2018 versions have the same versions).
*   Install necessary packages: `apt-get install autoconf binutils build-essential gawk gettext git libncurses5-dev libssl-dev libz-dev ncurses-term openssl sharutils subversion unzip`
*    Get the source `git clone https://github.com/openwrt/openwrt.git openwrt-git && cd $_`
*    We can use the latest config: `wget https://downloads.openwrt.org/snapshots/trunk/ar71xx/generic/config -O .config`
*    `make defconfig`
*    `make menuconfig`
    *    Target system is "Atheros AR7xxx/AR9xxx"
    *    Target profile is "GL AR150"
    *    Uncheck "Build the OpenWrt SDK"
*    Remove all modules: `sed 's/=m$/=n/' -i.bak .config`
*    Sanity check: `make prereq`
*    Build with: `script -c "time make -j4 V=s tools/install && date && time make -j4 V=s toolchain/install" build-$(date +'%FT%T').log`
*   Check out the packages from this git repository:
```
git clone https://github.com/mdhowe/openwrt.git ../openwrt-packages
ln -s ../../../../openwrt-packages/package/network/utils/monitoring-plugins package/network/utils/monitoring-plugins
ln -s ../../../../openwrt-packages/package/network/utils/nrpe package/network/utils/nrpe
```
*   `make oldconfig` and choose to make nrpe and monitoring-plugins as modules
*   `script -c "time make -j4 V=s package/nrpe/compile" -a nrpe-build-$(date +'%FT%T').log`
*   `script -c "time make -j4 V=s package/monitoring-plugins/compile" -a monitoring-plugins-build-$(date +'%FT%T').log`
*    Should find we have a bunch of ipk package files (the build and the dependencies): `ls -hgotr bin/packages/mips_24kc/base/`
*   **check** that they look sane - you want nrpe and monitoring-plugins, and perhaps libopenssl.  If you have libc (for example) things will probably break.
*    Copy across to the GL-AR150:
```
cd bin/packages/mips_24kc && tar -cvf /tmp/monitoring-packages.tar base/
scp /tmp/monitoring-packages.tar root@gl-ar150.domain:
```
*    Then, on the GL-AR150, install and enable nrpe:
```
tar xvf monitoring-packages.tar
opkg install base/*.ipk
/etc/init.d/nagios-nrpe enable
/etc/init.d/nagios-nrpe start
```
*   Add your source host to `allowed_hosts` in `/etc/nrpe.cfg` and `/etc/init.d/nagios-nrpe restart`
*   Confirm nrpe is running: `netstat -lnp | grep 5666`
*   Check from a client:
```
% /usr/lib/nagios/plugins/check_nrpe -H 192.168.8.1 -c check_load
OK - load average: 0.10, 0.39, 0.23|load1=0.100;15.000;30.000;0; load5=0.390;10.000;25.000;0; load15=0.230;5.000;20.000;0;
```
