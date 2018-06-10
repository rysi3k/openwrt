#
# Copyright (C) 2007-2016 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=nrpe
PKG_VERSION:=3.2.1
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://github.com/NagiosEnterprises/nrpe.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_VERSION:=86a77fe3a9cbfae990adc578834fd537f4fa8557
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_LICENSE:=GPL-2.0
PKG_INSTALL:=1

include $(INCLUDE_DIR)/package.mk

define Package/nrpe
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Nagios Remote Plugin Executor Server
  URL:=https://github.com/NagiosEnterprises/nrpe
  DEPENDS:=+libopenssl
  USERID:=nagios:nagios
  MAINTAINER:=Michael Howe <michael@michaelhowe.org>
endef

define Package/nrpe/description
	This program runs as a background process on the remote host and processes
	command execution requests from the check_nrpe plugin on the Nagios host.
endef

CONFIGURE_ARGS += \
	--with-ssl=$(STAGING_DIR)/usr \
	--with-ssl-lib=$(STAGING_DIR)/usr/lib \
	--with-wrap=no \
	--with-pluginsdir=/usr/libexec/nagios

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) nrpe
endef

define Package/nrpe/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_DIR) $(1)/etc
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/openwrt-init $(1)/etc/init.d/nagios-$(PKG_NAME)
	$(CP) $(PKG_BUILD_DIR)/src/nrpe $(1)/usr/sbin
	$(CP) $(PKG_BUILD_DIR)/sample-config/nrpe.cfg $(1)/etc/nrpe.cfg
endef

define Package/nrpe/conffiles
	/etc/nrpe.cfg
endef

$(eval $(call BuildPackage,nrpe))
