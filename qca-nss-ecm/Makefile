include $(TOPDIR)/rules.mk

PKG_NAME:=qca-nss-ecm
PKG_RELEASE:=1

# Source: https://git.codelinaro.org/clo/qsdk/oss/lklm/qca-nss-ecm.git @ e978478f5dba749d412e144b8f3e7b152687d893
# (tag NHSS.QSDK.14.0.r9-00025-O, the newest QSDK tag; pre-generated 'git archive' of the commit).
# ECM is the only one of the three that moved between r8 and r9; every r9 tag
# from r9-00007-O onward resolves to this commit.
# Fetched from our release mirror so the download is deterministic and hash-verified;
# git-proto regenerates the zstd archive locally and its hash varies by toolchain.
PKG_SOURCE_DATE:=2026-07-10
PKG_SOURCE_VERSION:=e978478f5dba749d412e144b8f3e7b152687d893
QSDK_VERSION:=14.0
PKG_VERSION:=$(QSDK_VERSION).$(subst -,.,$(PKG_SOURCE_DATE))~$(PKG_SOURCE_VERSION)
PKG_SOURCE:=$(PKG_NAME)-$(subst ~,.,$(PKG_VERSION)).tar.zst
PKG_SOURCE_URL:=https://github.com/JuliusBairaktaris/nss-packages/releases/download/nss-source-tarballs
PKG_HASH:=31539421a42aea1a6d5bdc806754414e9518a6d6e1d4870174345064ca03ba64

PKG_BUILD_PARALLEL:=1
PKG_FLAGS:=nonshared
PKG_BUILD_FLAGS:=gc-sections lto

PKG_CONFIG_DEPENDS:= \
	CONFIG_NSS_DRV_PPPOE_ENABLE \
	CONFIG_NSS_FIRMWARE_VERSION_11_4 \
	CONFIG_NSS_FIRMWARE_VERSION_12_5 \
	CONFIG_PACKAGE_kmod-ipsec \
	CONFIG_PACKAGE_kmod-nat46 \
	CONFIG_PACKAGE_kmod-pppoe \
	CONFIG_PACKAGE_kmod-qca-mcs \
	CONFIG_PACKAGE_kmod-qca-nss-drv \
	CONFIG_PACKAGE_kmod-qca-nss-drv-igs

include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/package.mk

define KernelPackage/qca-nss-ecm
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Network Support
  DEPENDS:=@(TARGET_qualcommbe||TARGET_qualcommax||TARGET_ipq806x) \
		   +ethtool \
		   +kmod-nf-conntrack \
		   +@(TARGET_qualcommax||TARGET_ipq806x):NSS_DRV_IPV6_ENABLE \
		   +@(TARGET_qualcommax||TARGET_ipq806x):NSS_DRV_VIRT_IF_ENABLE \
		   +@(TARGET_qualcommax||TARGET_ipq806x):NSS_DRV_PPPOE_ENABLE \
		   +PACKAGE_kmod-qca-nss-drv:kmod-qca-nss-drv \
		   +PACKAGE_kmod-qca-mcs:kmod-qca-mcs \
		   +PACKAGE_kmod-nat46:kmod-nat46 \
		   +kmod-ppp \
		   +kmod-pppoe
  TITLE:=QCA NSS Enhanced Connection Manager (ECM)
  FILES:=$(PKG_BUILD_DIR)/ecm.ko
  KCONFIG:= \
	CONFIG_NF_CONNTRACK_DSCPREMARK_EXT=y \
	CONFIG_NF_CONNTRACK_EVENTS=y
endef

define KernelPackage/qca-nss-ecm/Description
This package contains the QCA NSS Enhanced Connection Manager
endef

define KernelPackage/qca-nss-ecm/conffiles
/etc/config/ecm
endef

define KernelPackage/qca-nss-ecm/install
	$(INSTALL_DIR) $(1)/etc/init.d $(1)/usr/bin $(1)/lib/netifd/offload $(1)/etc/config $(1)/etc/uci-defaults $(1)/etc/hotplug.d/net
	$(INSTALL_BIN) ./files/qca-nss-ecm.init $(1)/etc/init.d/qca-nss-ecm
	$(INSTALL_BIN) ./files/ecm_dump.sh $(1)/usr/bin/
	$(INSTALL_DATA) ./files/disable_offloads.sh $(1)/lib/netifd/offload
	$(INSTALL_DATA) ./files/on-demand-down $(1)/lib/netifd/offload/on-demand-down
	$(INSTALL_DATA) ./files/qca-nss-ecm.uci $(1)/etc/config/ecm
	$(INSTALL_DATA) ./files/qca-nss-ecm.defaults $(1)/etc/uci-defaults/99-qca-nss-ecm
	$(INSTALL_DATA) ./files/disable_offloads.hotplug $(1)/etc/hotplug.d/net/99-disable_offloads
endef

EXTRA_CFLAGS+= \
		 -I$(STAGING_DIR)/usr/include/qca-nss-drv \
		 -I$(STAGING_DIR)/usr/include/qca-mcs \
		 -I$(STAGING_DIR)/usr/include/nat46

ifneq (, $(findstring $(CONFIG_TARGET_BOARD), "qualcommax" "ipq60xx"))
ECM_MAKE_OPTS+= ECM_NON_PORTED_SUPPORT_ENABLE=y \
			   ECM_INTERFACE_VLAN_ENABLE=y \
			   ECM_CLASSIFIER_MARK_ENABLE=y \
			   ECM_CLASSIFIER_DSCP_ENABLE=y \
			   ECM_CLASSIFIER_PCC_ENABLE=n
endif

ifneq ($(CONFIG_PACKAGE_kmod-qca-nss-drv),)
ECM_MAKE_OPTS+=ECM_FRONT_END_NSS_ENABLE=y
endif

# Disable ECM IPv6 support when global IPv6 support is disabled.
ifneq ($(CONFIG_IPV6),)
ECM_MAKE_OPTS+=ECM_IPV6_ENABLE=y
endif

# Disable ECM Bridge VLAN Filtering support when global kernel config is disabled
ifneq ($(CONFIG_BRIDGE_VLAN_FILTERING),)
ECM_MAKE_OPTS+=ECM_BRIDGE_VLAN_FILTERING_ENABLE=y
endif

ifneq ($(CONFIG_PACKAGE_kmod-qca-nss-drv-igs),)
ECM_MAKE_OPTS+=ECM_CLASSIFIER_DSCP_IGS=y
endif

ifneq ($(CONFIG_PACKAGE_kmod-qca-mcs),)
ECM_MAKE_OPTS+=ECM_MULTICAST_ENABLE=y
endif

ifneq ($(CONFIG_PACKAGE_kmod-nat46),)
ECM_MAKE_OPTS+=ECM_INTERFACE_MAP_T_ENABLE=y
endif

ifneq ($(CONFIG_PACKAGE_kmod-ipsec),)
ECM_MAKE_OPTS+=ECM_INTERFACE_IPSEC_ENABLE=y
endif

ifneq ($(CONFIG_PACKAGE_kmod-pppoe),)
ECM_MAKE_OPTS+=ECM_INTERFACE_PPPOE_ENABLE=y \
			   ECM_INTERFACE_PPP_ENABLE=y
else
ECM_MAKE_OPTS+=ECM_INTERFACE_PPPOE_ENABLE=n \
			   ECM_INTERFACE_PPP_ENABLE=n
endif

# The EDMA-rework kernel carries only the PPPoE ppp hooks (0961); the
# PPTP/L2TP kernel hooks (qosmio 0603-2/0603-3) are not ported, so
# these interface types must stay off regardless of selected kmods.
ECM_MAKE_OPTS+=ECM_INTERFACE_PPTP_ENABLE=n \
			   ECM_INTERFACE_L2TPV2_ENABLE=n

# GRE offload is not supported on this tree - never validated here, so it is
# forced off rather than switched on by whichever kmods happen to be selected.
ECM_MAKE_OPTS+=ECM_INTERFACE_GRE_TAP_ENABLE=n \
			   ECM_INTERFACE_GRE_TUN_ENABLE=n

# 6RD/SIT and IPIP6 offload are not supported on this tree: nothing here has
# been validated against them, and an interface type that is enabled but never
# exercised is worse than one that is off - it changes how ECM classifies
# flows without anyone having checked the result. Forced off regardless of
# kmod-sit / kmod-ip6-tunnel, the same way bonding is below.
ECM_MAKE_OPTS+=ECM_INTERFACE_SIT_ENABLE=n \
			   ECM_INTERFACE_TUNIPIP6_ENABLE=n

# The EDMA-rework kernel does not carry the QCA bonding hooks
# (bond_get_tx_dev), so LAG offload must stay off regardless of
# kmod-bonding.
ECM_MAKE_OPTS+=ECM_INTERFACE_BOND_ENABLE=n

ifneq ($(CONFIG_NSS_FIRMWARE_VERSION_11_4),)
EXTRA_CFLAGS+=-DNSS_FIRMWARE_VERSION_11_4
endif

define Build/InstallDev
	mkdir -p $(1)/usr/include/qca-nss-ecm
	$(CP) $(PKG_BUILD_DIR)/exports/* $(1)/usr/include/qca-nss-ecm
endef

ifeq ($(CONFIG_TARGET_BOARD), "qualcommax")
    SOC:=$(CONFIG_TARGET_SUBTARGET)
endif

define Build/Compile
	+$(MAKE) -C "$(LINUX_DIR)" $(strip $(ECM_MAKE_OPTS)) \
		CROSS_COMPILE="$(TARGET_CROSS)" \
		ARCH="$(LINUX_KARCH)" \
		M="$(PKG_BUILD_DIR)" \
		NOSTDINC_FLAGS="$(KERNEL_NOSTDINC_FLAGS) $(EXTRA_CFLAGS)" \
		CFLAGS_MODULE="-Wno-error" \
		SoC=$(SOC) \
		$(KERNEL_MAKE_FLAGS) \
		$(PKG_JOBS) \
		modules
endef

$(eval $(call KernelPackage,qca-nss-ecm))
