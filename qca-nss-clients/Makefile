include $(TOPDIR)/rules.mk

PKG_NAME:=qca-nss-clients
PKG_RELEASE:=8

# Source: https://git.codelinaro.org/clo/qsdk/oss/lklm/nss-clients.git @ 51be82d (pre-generated 'git archive' of the commit).
# Fetched from our release mirror so the download is deterministic and hash-verified;
# git-proto regenerates the zstd archive locally and its hash varies by toolchain.
PKG_SOURCE_DATE:=2024-09-11
PKG_SOURCE_VERSION:=51be82d
QSDK_VERSION:=12.5
PKG_VERSION:=$(QSDK_VERSION).$(subst -,.,$(PKG_SOURCE_DATE))~$(PKG_SOURCE_VERSION)
PKG_SOURCE:=$(PKG_NAME)-$(subst ~,.,$(PKG_VERSION)).tar.zst
PKG_SOURCE_URL:=https://github.com/JuliusBairaktaris/nss-packages/releases/download/nss-source-tarballs
PKG_HASH:=876c3898cb4723fccd4a1c870324724dd7c4061ac89ec88db2226fd749c18b38

PKG_BUILD_PARALLEL:=1

PKG_CONFIG_DEPENDS:=\
	CONFIG_NSS_DRV_IGS_ENABLE \
	CONFIG_NSS_DRV_PPPOE_ENABLE \
	CONFIG_NSS_DRV_SHAPER_ENABLE

include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/package.mk

ifeq ($(CONFIG_TARGET_SUBTARGET), "ipq807x")
  SOC="ipq807x_64"
  subtarget:=$(CONFIG_TARGET_SUBTARGET)
else ifeq ($(CONFIG_TARGET_SUBTARGET), "ipq60xx")
  SOC="ipq60xx_64"
  subtarget:=$(CONFIG_TARGET_SUBTARGET)
else ifeq ($(CONFIG_TARGET_SUBTARGET), "ipq50xx")
  SOC="ipq50xx_64"
  subtarget:=$(CONFIG_TARGET_SUBTARGET)
else
  SOC=$(CONFIG_TARGET_BOARD)
endif

define KernelPackage/qca-nss-drv-pppoe
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Network Devices
  TITLE:=NSS connection manager for PPPoE protocol
  DEPENDS:=@(TARGET_qualcommax||TARGET_ipq806x) \
	   +kmod-qca-nss-drv \
	   +@NSS_DRV_PPPOE_ENABLE \
	   +kmod-ppp \
	   +kmod-pppoe
  FILES:=$(PKG_BUILD_DIR)/pppoe/qca-nss-pppoe.ko
endef

define KernelPackage/qca-nss-drv-pppoe/Description
Kernel modules for NSS connection manager - Support for PPPoE
endef

define KernelPackage/qca-nss-drv-qdisc
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Network Devices
  TITLE:=Qdisc for configuring shapers in NSS
  DEPENDS:=@(TARGET_qualcommax||TARGET_ipq806x) \
	   +kmod-qca-nss-drv \
	   +@NSS_DRV_SHAPER_ENABLE \
	   +@NSS_DRV_IGS_ENABLE
  FILES:=$(PKG_BUILD_DIR)/nss_qdisc/qca-nss-qdisc.ko
  KCONFIG:=CONFIG_NET_CLS_ACT=y
endef

define KernelPackage/qca-nss-drv-qdisc/Description
Linux qdisc that aids in configuring shapers in the NSS
endef

define KernelPackage/qca-nss-drv-igs
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Network Devices
  TITLE:=NSS connection manager to perform ingress shaping
  DEPENDS:=@(TARGET_qualcommax||TARGET_ipq806x) \
	   +kmod-qca-nss-drv \
	   +@NSS_DRV_IGS_ENABLE \
	   +kmod-qca-nss-drv-qdisc \
	   +kmod-ifb \
	   +kmod-nf-conntrack \
	   +kmod-sched-core
  FILES:=$(PKG_BUILD_DIR)/nss_qdisc/igs/act_nssmirred.ko
endef

define KernelPackage/qca-nss-drv-igs/description
Linux action that helps in offloading traffic to an IFB interface to perform ingress shaping.
endef

define KernelPackage/qca-nss-drv-bridge-mgr
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Network Devices
  TITLE:=NSS bridge manager (hardware bridge offload)
  DEPENDS:=@(TARGET_qualcommax_ipq807x||TARGET_qualcommax_ipq60xx) \
	   +kmod-qca-nss-drv \
	   +@NSS_DRV_BRIDGE_ENABLE \
	   +kmod-qca-ppe-nss
  FILES:=$(PKG_BUILD_DIR)/bridge/qca-nss-bridge-mgr.ko
endef

define KernelPackage/qca-nss-drv-bridge-mgr/Description
Offloads Linux-bridge L2 forwarding into the NSS firmware by binding the
bridge's qca_ppe member ports to a firmware bridge interface. No AUTOLOAD:
loaded at runtime after the NSS data plane is up (see nss-up).
endef

# Vendor code is not warning-clean; openwrt main enables
# CONFIG_KERNEL_WERROR which leaks -Werror into external modules.
EXTRA_CFLAGS+= -Wno-error

EXTRA_CFLAGS+= \
	-I$(STAGING_DIR)/usr/include/qca-nss-drv

# Build individual packages if selected
ifneq ($(CONFIG_PACKAGE_kmod-qca-nss-drv-qdisc),)
NSS_CLIENTS_MAKE_OPTS+=qdisc=y
# PPE shaper mode and NSS-bridge qdisc attach need qca-ssdk and
# bridge-mgr, neither of which exists on the EDMA stack. The NSS-fw
# accel mode (accel_mode 0) is unaffected.
NSS_CLIENTS_MAKE_OPTS+=NSS_QDISC_PPE_ENABLE=n
endif

ifneq ($(CONFIG_PACKAGE_kmod-qca-nss-drv-igs),)
NSS_CLIENTS_MAKE_OPTS+=igs=y
endif

ifneq ($(CONFIG_PACKAGE_kmod-qca-nss-drv-pppoe),)
NSS_CLIENTS_MAKE_OPTS+=pppoe=y
endif

ifneq ($(CONFIG_PACKAGE_kmod-qca-nss-drv-bridge-mgr),)
NSS_CLIENTS_MAKE_OPTS+=bridge-mgr=y
endif

EXTRA_CFLAGS+= \
	-Wno-missing-prototypes \
	-Wno-missing-declarations \
	-Wno-empty-body

define Build/Compile
	+$(MAKE) -C "$(LINUX_DIR)" $(strip $(NSS_CLIENTS_MAKE_OPTS)) \
	  CROSS_COMPILE="$(TARGET_CROSS)" \
	  ARCH="$(LINUX_KARCH)" \
	  M="$(PKG_BUILD_DIR)" \
	  EXTRA_CFLAGS="$(EXTRA_CFLAGS) -include $(PKG_BUILD_DIR)/compat.h" \
	  SoC=$(SOC) \
	  $(KERNEL_MAKE_FLAGS) \
	  $(PKG_JOBS) \
	  modules
endef

$(eval $(call KernelPackage,qca-nss-drv-pppoe))
$(eval $(call KernelPackage,qca-nss-drv-qdisc))
$(eval $(call KernelPackage,qca-nss-drv-igs))
$(eval $(call KernelPackage,qca-nss-drv-bridge-mgr))
