
include ../../config.mk
include ../wvports.mk

define unpack
	ln -s openssl-0.9.8zg $(BUILDDIR)/openssl
endef

# for 'make diff'
EXCLUDE=opensslconf.h buildinf.h \
	*.s *.bak *.0 *.so *.so.* *.a *.pc \
	openssl Makefile *test


ifeq ($(WV_BUILD_MINGW),1)

# MINGW32 build

define prepare
	cd build/openssl && ./Configure mingw
endef

define compile
	make -C build/openssl \
		CC="$(CC)" RANLIB="$(RANLIB)" AR="$(ARRC)" \
		CFLAG="-DL_ENDIAN -DDSO_WIN32 -O3 -march=i486 -Wall -DBN_ASM -DMD5_ASM -DSHA1_ASM -DOPENSSL_BN_ASM_PART_WORDS -DOPENSSL_NO_CAMELLIA -DOPENSSL_NO_RC5 -DOPENSSL_NO_MDC2 -DOPENSSL_NO_KRB5 -DOPENSSL_NO_DYNAMIC_ENGINE -DNO_WINDOWS_BRAINDEATH"
endef


else

# Normal Unix build

define prepare
	cd build/openssl && ./config -DPURIFY -shared -g
endef

define compile
	make -C build/openssl
endef


endif

