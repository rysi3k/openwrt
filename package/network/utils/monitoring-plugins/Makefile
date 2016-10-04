#
# Copyright (C) 2008-2016 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=monitoring-plugins
PKG_VERSION:=2.1.2
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://github.com/monitoring-plugins/monitoring-plugins.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_VERSION:=5f1371fb6fec0747dde6ced777bbac18081c25f4
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_LICENSE:=GPL-3.0
PKG_INSTALL:=1

include $(INCLUDE_DIR)/package.mk

define Package/monitoring-plugins
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Monitoring Plugins
  URL:=https://www.monitoring-plugins.org
  DEPENDS:=+libopenssl +libpthread
  MAINTAINER:=Christian Kujau <openwrt@nerdbynature.de>
endef
  
define Package/nrpe/description
	Plugins for Nagios compatible monitoring systems like Naemon or Icinga.
endef

define Build/Prepare
	$(Build/Prepare/Default)
	(cd $(PKG_BUILD_DIR) && ./tools/setup)
endef

CONFIGURE_ARGS += \
	--without-dbi \
	--without-ldap \
	--without-mysql \
	--without-pgsql \
	--without-radius \
	--with-openssl=$(STAGING_DIR)/usr \
	--with-ping-command="/bin/ping -w %d -c %d %s" \
	--with-ping6-command="/bin/ping6 -w %d -c %d %s" \
	--with-ps-command="/bin/ps w" \
	--with-ps-cols="6" \
	--with-ps-format="%d %s %d %s %s %n" \
	--with-ps-varlist="&procpid$(comma)&procuid$(comma)&procvsz$(comma)procstat$(comma)procprog$(comma)&pos" \
	--with-uptime-command="/usr/bin/uptime" \
	--with-ssh-command="/usr/bin/ssh" \
	--with-nslookup-command="/usr/bin/nslookup"


define Package/monitoring-plugins/install
	$(INSTALL_DIR) $(1)/usr/libexec/nagios
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/* $(1)/usr/libexec/nagios/
endef

$(eval $(call BuildPackage,monitoring-plugins))
