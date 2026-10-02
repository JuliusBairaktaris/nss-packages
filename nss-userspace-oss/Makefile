include $(TOPDIR)/rules.mk

PKG_NAME:=nss-userspace-oss
PKG_RELEASE:=2

# Source: https://git.codelinaro.org/clo/qsdk/oss/nss-userspace.git @ 9584ff2 (pre-generated 'git archive' of the commit).
# Head of NHSS.RDK.14.0.r4, the newest branch this repository carries.
# Fetched from our release mirror so the download is deterministic and hash-verified;
# git-proto regenerates the zstd archive locally and its hash varies by toolchain.
PKG_SOURCE_DATE:=2026-06-24
PKG_SOURCE_VERSION:=9584ff2
QSDK_VERSION:=14.0
PKG_VERSION:=$(QSDK_VERSION).$(subst -,.,$(PKG_SOURCE_DATE))~$(PKG_SOURCE_VERSION)
PKG_SOURCE:=$(PKG_NAME)-$(subst ~,.,$(PKG_VERSION)).tar.zst
PKG_SOURCE_URL:=https://github.com/JuliusBairaktaris/nss-packages/releases/download/nss-source-tarballs
PKG_HASH:=616e852c14acde8fe05dfc9b661b4ee7988d5d76bbd0533386afe0512efce70a

PKG_BUILD_PARALLEL:=1
PKG_FLAGS:=nonshared
PKG_BUILD_FLAGS:=gc-sections lto

PKG_CONFIG_DEPENDS:= \
	CONFIG_NSS_NLCAPWAP_ENABLE \
	CONFIG_NSS_NLDTLS_ENABLE \
	CONFIG_NSS_NLGRE_REDIR_ENABLE \
	CONFIG_NSS_NLIPSEC_ENABLE \
	CONFIG_NSS_NLQRFS_ENABLE \
	CONFIG_NSS_NLUDP_ST_ENABLE
	
include $(INCLUDE_DIR)/package.mk

define Package/nss-userspace
  TITLE:=Userspace utilities for NSS
endef

define Package/libnl-nss
$(call Package/nss-userspace)
  SECTION:=Libs
  CATEGORY:=Libraries
  TITLE += (libnl-nss)
  DEPENDS:=+libpthread +libnl-tiny +kmod-qca-nss-drv-netlink
endef

define Package/libnl-nss/description
 A framework in the userspace that establishes communication between userspace applications and the kernel.
endef

define Package/nssinfo
$(call Package/nss-userspace)
  SECTION:=Utils
  CATEGORY:=Utilities
  TITLE += (nssinfo)
	DEPENDS:=+libncurses +libnl-nss
endef

define Package/nssinfo/description
 A userspace utility for fetching stats from NSS.
endef

TARGET_CFLAGS += $(FPIC) -D_GNU_SOURCE=1

TARGET_CPPFLAGS:= \
		-I$(STAGING_DIR)/usr/include/qca-nss-clients \
		-I$(STAGING_DIR)/usr/include/qca-nss-drv \
		-I$(STAGING_DIR)/usr/include/libnl-tiny \
		-I$(STAGING_DIR)/usr/include/libnl-nss

ifneq ($(CONFIG_PACKAGE_libnl-nss),)
MAKE_FLAGS+=BUILD_LIBNSS=y

ifdef CONFIG_NSS_NLUDP_ST_ENABLE
MAKE_FLAGS+=udp_st=y
endif

ifdef CONFIG_NSS_NLQRFS_ENABLE
MAKE_FLAGS+=qrfs=y
endif

ifdef CONFIG_NSS_NLGRE_REDIR_ENABLE
MAKE_FLAGS+=gre_redir=y
endif

ifdef CONFIG_NSS_NLIPSEC_ENABLE
MAKE_FLAGS+=ipsec=y
endif

ifdef CONFIG_NSS_NLDTLS_ENABLE
MAKE_FLAGS+=dtls=y
endif

ifdef CONFIG_NSS_NLCAPWAP_ENABLE
MAKE_FLAGS+=capwap=y
endif
endif

ifneq ($(CONFIG_PACKAGE_nssinfo),)
MAKE_FLAGS+=BUILD_NSSINFO=y
endif

define Build/libnl-nss/InstallDev
	$(INSTALL_DIR) $(STAGING_DIR)/usr/include/libnl-nss
	$(CP) $(PKG_BUILD_DIR)/lib/include/* $(STAGING_DIR)/usr/include/libnl-nss
endef

define Package/libnl-nss/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/obj/libnl-nss.so $(1)/usr/lib
endef

define Package/nssinfo/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/nssinfo/obj/nssinfo $(1)/usr/sbin/
endef

$(eval $(call BuildPackage,nssinfo))
$(eval $(call BuildPackage,libnl-nss))
