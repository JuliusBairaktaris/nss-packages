#
# Copyright (C) 2022 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=nss-firmware
PKG_VERSION:=2025.05.01
PKG_HASH:=10a4b1e69470db150915cb063525436494b8ae4eebb8b50ea6ad894082d7abb0
PKG_SOURCE_URL:=https://github.com/qosmio/qca-sdk-nss-fw/releases/download/v$(PKG_VERSION)
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.zst
PKG_RELEASE:=1

PKG_LICENSE_FILES:=LICENSE.md

PKG_MAINTAINER:=Sean K <datapronix@protonmail.com>

PKG_CONFIG_DEPENDS:= \
	CONFIG_NSS_FIRMWARE_VERSION_11_4 \
	CONFIG_NSS_FIRMWARE_VERSION_12_5

include $(INCLUDE_DIR)/package.mk

RSTRIP:=:
STRIP:=:

NSS_PROFILE:=R

define Package/nss-firmware-common
	TITLE:=NSS firmware
	SECTION:=firmware
	CATEGORY:=Firmware
	URL:=$(PKG_SOURCE_URL)
	DEPENDS:=@TARGET_qualcommax
endef

define Package/nss-firmware
$(call Package/nss-firmware-common,$(1))
	DEPENDS+= +TARGET_qualcommax_ipq807x:nss-firmware-ipq807x \
		  +TARGET_qualcommax_ipq60xx:nss-firmware-ipq60xx
endef

define Package/nss-firmware-ipq807x
$(call Package/nss-firmware-common,$(1))
	DEPENDS+=@TARGET_qualcommax_ipq807x
	IPQ_PLATFORM=IPQ8074
	NSS_SOC:=HK
endef

define Package/nss-firmware-ipq60xx
$(call Package/nss-firmware-common,$(1))
	DEPENDS+=@TARGET_qualcommax_ipq60xx
	IPQ_PLATFORM=IPQ6018
	NSS_SOC:=CP
endef

define Package/nss-firmware/config
  menu "NSS Firmware Version"

  comment "Select NSS firmware version"

  choice
    prompt "Version"
    default NSS_FIRMWARE_VERSION_12_5

    config NSS_FIRMWARE_VERSION_12_5
          bool "NSS Firmware 12.5 Release 210"
          help
               The default firmware line. Does not accept an 802.11s
               mesh interface.

    config NSS_FIRMWARE_VERSION_11_4
          bool "NSS Firmware 11.4.0.5 Release 6"
          help
               The last firmware line with 802.11s mesh support,
               needed for mesh offload. Every firmware newer than 11.4
               rejects mesh interface allocation.
  endchoice
  endmenu
endef

# Both lines are installed as qca-nss<N>-retail-<line>.bin, and the plain
# name points at the line selected above.
NSS_LINE_12.5:=210 QCA_Networking_2024.SPF_12.5/ED1
NSS_LINE_11.4:=6 QCA_Networking_2021.SPF_11.4/CS
NSS_LINE:=$(if $(CONFIG_NSS_FIRMWARE_VERSION_11_4),11.4,12.5)

define nss-firmware-line
	$(INSTALL_DIR) $(PKG_BUILD_DIR)/$(IPQ_PLATFORM)/$(2)
	$(TAR) --strip-components=1 -C $(PKG_BUILD_DIR)/$(IPQ_PLATFORM)/$(2) -xf \
		$(PKG_BUILD_DIR)/$(word 2,$(NSS_LINE_$(2)))/$(IPQ_PLATFORM).ATH.$(2)/BIN-NSS*.$(2)*-$(word 1,$(NSS_LINE_$(2)))*$(NSS_PROFILE).tar.bz2
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/$(IPQ_PLATFORM)/$(2)/retail_router0.bin \
		$(1)/lib/firmware/qca-nss0-retail-$(2).bin
	$(if $(filter HK,$(NSS_SOC)),$(INSTALL_DATA) \
		$(PKG_BUILD_DIR)/$(IPQ_PLATFORM)/$(2)/retail_router1.bin \
		$(1)/lib/firmware/qca-nss1-retail-$(2).bin)
endef

define Build/Compile
endef

define Package/nss-firmware-common/install
	$(INSTALL_DIR) $(1)/lib/firmware
	$(call nss-firmware-line,$(1),12.5)
	$(call nss-firmware-line,$(1),11.4)
	$(LN) qca-nss0-retail-$(NSS_LINE).bin $(1)/lib/firmware/qca-nss0-retail.bin
	$(if $(filter HK,$(NSS_SOC)),$(LN) qca-nss1-retail-$(NSS_LINE).bin \
		$(1)/lib/firmware/qca-nss1-retail.bin)
endef

define Package/nss-firmware-ipq807x/install
	$(call Package/nss-firmware-common/install,$(1))
endef

define Package/nss-firmware-ipq60xx/install
	$(call Package/nss-firmware-common/install,$(1))
endef

define Package/nss-firmware/install
	true
endef

$(eval $(call BuildPackage,nss-firmware-ipq807x))
$(eval $(call BuildPackage,nss-firmware-ipq60xx))
$(eval $(call BuildPackage,nss-firmware))
