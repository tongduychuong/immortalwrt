#
# Copyright (C) 2006-2017 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=wolfssl
PKG_VERSION:=5.7.6-stable
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://github.com/wolfSSL/wolfssl/archive/v$(PKG_VERSION)
PKG_HASH:=52b1e439e30d1ed8162a16308a8525a862183b67aa30373b11166ecbab000d63

PKG_FIXUP:=libtool libtool-abiver
PKG_INSTALL:=1
PKG_BUILD_FLAGS:=no-mips16 lto
PKG_BUILD_PARALLEL:=1
PKG_LICENSE:=GPL-2.0-or-later
PKG_LICENSE_FILES:=LICENSING COPYING
PKG_MAINTAINER:=Eneas U de Queiroz <cotequeiroz@gmail.com>
PKG_CPE_ID:=cpe:/a:wolfssl:wolfssl

PKG_CONFIG_DEPENDS:=\
	CONFIG_WOLFSSL_HAS_AES_CCM \
	CONFIG_WOLFSSL_HAS_ARC4 \
	CONFIG_WOLFSSL_HAS_CERTGEN \
	CONFIG_WOLFSSL_HAS_CHACHA_POLY \
	CONFIG_WOLFSSL_HAS_DH \
	CONFIG_WOLFSSL_HAS_DTLS \
	CONFIG_WOLFSSL_HAS_ECC25519 \
	CONFIG_WOLFSSL_HAS_ECC448 \
	CONFIG_WOLFSSL_HAS_OCSP \
	CONFIG_WOLFSSL_HAS_OPENVPN CONFIG_WOLFSSL_ALT_NAMES \
	CONFIG_WOLFSSL_HAS_SESSION_TICKET \
	CONFIG_WOLFSSL_HAS_TLSV10 \
	CONFIG_WOLFSSL_HAS_TLSV13 \
	CONFIG_WOLFSSL_HAS_WPAS

PKG_ABI_VERSION:=$(patsubst %-stable,%,$(PKG_VERSION)).$(call version_abbrev,$(call confvar,$(PKG_CONFIG_DEPENDS)))

PKG_CONFIG_DEPENDS+=\
	CONFIG_PACKAGE_libwolfssl-benchmark \
	CONFIG_WOLFSSL_HAS_AFALG \
	CONFIG_WOLFSSL_HAS_DEVCRYPTO_AES \
	CONFIG_WOLFSSL_HAS_DEVCRYPTO_CBC \
	CONFIG_WOLFSSL_HAS_DEVCRYPTO_FULL

include $(INCLUDE_DIR)/package.mk

DISABLE_NLS:=

define Package/libwolfssl/Default
  SECTION:=libs
  SUBMENU:=SSL
  CATEGORY:=Libraries
  URL:=http://www.wolfssl.com/
endef

define Package/libwolfssl
$(call Package/libwolfssl/Default)
  TITLE:=wolfSSL library
  MENU:=1
  PROVIDES:=libcyassl
  DEPENDS:=+WOLFSSL_HAS_DEVCRYPTO:kmod-cryptodev +WOLFSSL_HAS_AFALG:kmod-crypto-user
  ABI_VERSION:=$(PKG_ABI_VERSION)
  VARIANT:=regular
  DEFAULT_VARIANT:=1
  CONFLICTS:=libwolfsslcpu-crypto
endef

define Package/libwolfssl/description
wolfSSL (formerly CyaSSL) is an SSL library optimized for small
footprint, both on disk and for memory use.
endef

define Package/libwolfssl/config
	source "$(SOURCE)/Config.in"
endef

define Package/libwolfsslcpu-crypto
$(call Package/libwolfssl/Default)
  TITLE:=wolfSSL library with AES CPU instructions
  PROVIDES:=libwolfssl libcyassl
  DEPENDS:=@((aarch64||x86_64)&&(m||!TARGET_bcm27xx))
  ABI_VERSION:=$(PKG_ABI_VERSION)
  VARIANT:=cpu-crypto
endef

define Package/libwolfssl-benchmark
$(call Package/libwolfssl/Default)
  TITLE:=wolfSSL Benchmark Utility
  DEPENDS:=libwolfssl
endef

define Package/libwolfsslcpu-crypto/description
$(call Package/libwolfssl/description)
This variant uses AES CPU instructions (Intel AESNI or ARMv8 Crypto Extension)
endef

define Package/libwolfsslcpu-crypto/config
    if TARGET_armsr && PACKAGE_libwolfsslcpu-crypto = y
	comment "You are about to build libwolfsslcpu-crypto into an armsr_64 image."
	comment "Ensure all of your installation targets support the Crypto Extension. "
	comment "Look for the 'aes' feature in /proc/cpuinfo. This library does not do "
	comment "run-time detection and will crash if the CPU does not support it.     "
    endif
    if TARGET_bcm27xx && PACKAGE_libwolfsslcpu-crypto
	comment "Beware that libwolfsslcpu-crypto will not run in a bcm27xx target.   "
    endif
endef

define Package/libwolfssl-benchmark/description
This is the wolfssl benchmark utility.
endef

TARGET_CFLAGS += \
	$(FPIC) \
	-fomit-frame-pointer \
	-DFP_MAX_BITS=8192 \
	$(if $(CONFIG_WOLFSSL_ALT_NAMES),-DWOLFSSL_ALT_NAMES)

# --enable-stunnel needed for OpenSSL API compatibility bits
CONFIGURE_ARGS += \
	--enable-reproducible-build \
	--enable-lighty \
	--enable-opensslall \
	--enable-opensslextra \
	--enable-sni \
	--enable-stunnel \
	--enable-altcertchains \
	--$(if $(CONFIG_PACKAGE_libwolfssl-benchmark),enable,disable)-crypttests \
	--disable-examples \
	--disable-jobserver \
	--$(if $(CONFIG_IPV6),enable,disable)-ipv6 \
	--$(if $(CONFIG_WOLFSSL_HAS_AES_CCM),enable,disable)-aesccm \
	--$(if $(CONFIG_WOLFSSL_HAS_CERTGEN),enable,disable)-certgen \
	--$(if $(CONFIG_WOLFSSL_HAS_CHACHA_POLY),enable,disable)-chacha \
	--$(if $(CONFIG_WOLFSSL_HAS_CHACHA_POLY),enable,disable)-poly1305 \
	--$(if $(CONFIG_WOLFSSL_HAS_DH),enable,disable)-dh \
	--$(if $(CONFIG_WOLFSSL_HAS_ARC4),enable,disable)-arc4 \
	--$(if $(CONFIG_WOLFSSL_HAS_TLSV10),enable,disable)-tlsv10 \
	--$(if $(CONFIG_WOLFSSL_HAS_TLSV13),enable,disable)-tls13 \
	--$(if $(CONFIG_WOLFSSL_HAS_SESSION_TICKET),enable,disable)-session-ticket \
	--$(if $(CONFIG_WOLFSSL_HAS_DTLS),enable,disable)-dtls \
	--$(if $(CONFIG_WOLFSSL_HAS_ECC25519),enable,disable)-curve25519 \
	--$(if $(CONFIG_WOLFSSL_HAS_ECC448),enable,disable)-curve448 \
	--$(if $(CONFIG_WOLFSSL_HAS_OPENVPN),enable,disable)-openvpn

define Package/libwolfsslcpu-crypto/preinst-aarch64
#!/bin/sh
exec >&2
printf "[libwolfsslcpu-crypto] Checking for Arm v8-A Cryptographic Extension support: "
if [ -n "$${IPKG_INSTROOT}" ]; then
    printf "...[offline]... "
    eval "$$(grep '^DISTRIB_TARGET=' "$${IPKG_INSTROOT}/etc/openwrt_release")"
    echo "$${DISTRIB_TARGET}" | grep '^bcm27xx/.*' > /dev/null && {
	echo "not supported"
	echo "Error: Target $${DISTRIB_TARGET} does not support Arm Cryptographic Extension."
	echo "Install the regular libwolfssl package instead of libwolfsslcpu-crypto."
	exit 1
    }
else
    grep -q '^Features.*\baes\b' /proc/cpuinfo || {
	echo "not supported"
	echo "Error: Arm v8-A Cryptographic Extension not supported."
	echo "Install the regular libwolfssl package instead of libwolfsslcpu-crypto."
	echo "Contents of /proc/cpuinfo:"
	cat /proc/cpuinfo
	exit 1
    }
fi
echo OK
exit 0
endef

ifeq ($(BUILD_VARIANT),regular)
CONFIGURE_ARGS += \
	--$(if $(CONFIG_WOLFSSL_HAS_AFALG),enable,disable)-afalg \
	--enable-devcrypto=$(if $(CONFIG_WOLFSSL_HAS_DEVCRYPTO_CBC),cbc\
			  ,$(if $(CONFIG_WOLFSSL_HAS_DEVCRYPTO_AES),aes\
			  ,$(if $(CONFIG_WOLFSSL_HAS_DEVCRYPTO_FULL),yes,no)))
else ifdef CONFIG_aarch64
    CONFIGURE_ARGS += --enable-armasm
    TARGET_CFLAGS:=$(TARGET_CFLAGS:-mcpu%=-mcpu%+crypto)
    Package/libwolfsslcpu-crypto/preinst=$(Package/libwolfsslcpu-crypto/preinst-aarch64)
else ifdef CONFIG_TARGET_x86_64
	CONFIGURE_ARGS += --enable-intelasm
endif

ifeq ($(CONFIG_WOLFSSL_HAS_OCSP),y)
CONFIGURE_ARGS += \
	--enable-ocsp --enable-ocspstapling --enable-ocspstapling2
endif

ifeq ($(CONFIG_WOLFSSL_HAS_WPAS),y)
CONFIGURE_ARGS += \
	--enable-wpas --enable-fortress --enable-fastmath
endif

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include $(1)/usr/lib/pkgconfig
	$(CP) $(PKG_INSTALL_DIR)/usr/include/* $(1)/usr/include/

	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libwolfssl.{so*,la} $(1)/usr/lib/
	ln -s libwolfssl.so $(1)/usr/lib/libcyassl.so
	ln -s libwolfssl.la $(1)/usr/lib/libcyassl.la

	$(CP) $(PKG_INSTALL_DIR)/usr/lib/pkgconfig/*.pc $(1)/usr/lib/pkgconfig
endef

define Package/libwolfssl/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libwolfssl.so.* $(1)/usr/lib/
endef

Package/libwolfsslcpu-crypto/install=$(Package/libwolfssl/install)

define Package/libwolfssl-benchmark/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(CP) $(PKG_BUILD_DIR)/wolfcrypt/benchmark/.libs/benchmark $(1)/usr/bin/wolfssl-benchmark
endef

$(eval $(call BuildPackage,libwolfssl))
$(eval $(call BuildPackage,libwolfsslcpu-crypto))
$(eval $(call BuildPackage,libwolfssl-benchmark))
