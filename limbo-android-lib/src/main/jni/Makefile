.PHONY: createdirs ndk qemu limbo

include android-config.mak

ifeq ($(NDK_ENV), windows)
	NDK_BUILD = ndk-build.cmd
else ifeq ($(NDK_ENV), windows-x86_64)
	NDK_BUILD = ndk-build.cmd
else
	NDK_BUILD = ndk-build
endif

ndkbuild=cd $(NDK_PROJECT_PATH) && $(NDK_BUILD) -j $(BUILD_THREADS) NDK_DEBUG=$(NDK_DEBUG) V=$(V) LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT)
ndkclean=cd $(NDK_PROJECT_PATH) && $(NDK_BUILD) clean NDK_DEBUG=$(NDK_DEBUG) V=$(V) LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT)
ndkcleanobj=rm -rf $(NDK_PROJECT_PATH)/obj/

ifeq ($(NDK_ENV), windows)
	ndkgdb=cd $(NDK_PROJECT_PATH) && ndk-gdb.cmd
else ifeq ($(NDK_ENV), windows-x86_64)
	ndkgdb=cd $(NDK_PROJECT_PATH) && ndk-gdb.cmd
else
	ndkgdb=cd $(NDK_PROJECT_PATH) && ndk-gdb
endif

CREATE_OBJDIR=mkdir -p $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/
CREATE_LIBSDIR=mkdir -p $(NDK_PROJECT_PATH)/libs/$(APP_ABI_DIR)/
DEL_LIBSDIR=rm -rf $(NDK_PROJECT_PATH)/libs/armeabi \
	../libs/armeabi-v7a \
	../libs/x86 ../libs/mips

INSTALL_LIBS=cp -f $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/*.so \
	../libs/$(APP_ABI_DIR)/

REMOVE_GDBSETUP=rm -f ../libs/$(APP_ABI_DIR)/gdb.setup
REMOVE_GDBSERVER=rm -f ../libs/$(APP_ABI_DIR)/gdbserver

COPY_ASSETS=cp -rf ../assets/* ../../../../limbo-android-x86/src/main/assets/

SYNC_LIBS=rsync -av --delete --delete-excluded --exclude libqemu-system-* ../libs/* ../jnilibs/

SYNC_LIBS_X86=rsync -av --include='libqemu-system-x86*.so' --include='*/' --exclude='*' ../libs/ ../jnilibs/
SYNC_LIBS_ARM=rsync -av --include='libqemu-system-arm*.so' --include='*/' --exclude='*' ../libs/ ../jnilibs/
SYNC_LIBS_SPARC=rsync -av --include='libqemu-system-sparc*.so' --include='*/' --exclude='*' ../libs/ ../jnilibs/
SYNC_LIBS_PPC=rsync -av --include='libqemu-system-ppc*.so' --include='*/' --exclude='*' ../libs/ ../jnilibs/

#RENAME_TUNCTL=mv -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/tunctl \
	$(NDK_PROJECT_PATH)/libs/$(APP_ABI)/libtunctl.so
#RENAME_PARPROUTED=mv -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/parprouted \
	$(NDK_PROJECT_PATH)/libs/$(APP_ABI)/libparprouted.so
	
#GDB needs the gdb.setup under ./libs & ./obj/local/armeabi
COPY_GDB_SETUP_LIBS=-cp -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/gdb.setup \
	$(NDK_PROJECT_PATH)/libs/gdb.setup
CREATE_LIBSDIR_ARMEABI=-mkdir -p $(NDK_PROJECT_PATH)/libs/armeabi
CREATE_OBJDIR_ARMEABI=-mkdir -p $(NDK_PROJECT_PATH)/obj/local/armeabi
COPY_GDB_SETUP_LIBS_ARMEABI=-cp -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/gdb.setup \
	$(NDK_PROJECT_PATH)/libs/armeabi/gdb.setup
COPY_GDB_SETUP_OBJ_ARMEABI=-cp -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/gdb.setup \
	$(NDK_PROJECT_PATH)/obj/local/armeabi/gdb.setup
APPEND_GDB_CMD_EXTRA=cat $(NDK_PROJECT_PATH)/jni/gdb.setup.extra >> \
	$(NDK_PROJECT_PATH)/libs/armeabi/gdb.setup; \
	cat $(NDK_PROJECT_PATH)/jni/gdb.setup.extra >> \
	$(NDK_PROJECT_PATH)/libs/$(APP_ABI)/gdb.setup;
	
ifneq ($(NDK_DEBUG),1)
	STRIP_LIBS=$(STRIP) --strip-unneeded  $(subst /cygdrive/c,c:,$(NDK_PROJECT_PATH))/libs/$(APP_ABI_DIR)/*.so
endif
	
#debug is default
# use "make release" for production version
all: debug
	
	
createdirs:
	$(CREATE_OBJDIR)
	$(DEL_LIBSDIR)
	$(CREATE_LIBSDIR)

ndk:
	$(ndkbuild) 

ndkclean:
	$(ndkclean)
	$(ndkcleanobj) 
	
qemu:
	$(MAKE) -f $(LIMBO_JNI_ROOT)/qemu.mak all NDK_DEBUG=$(NDK_DEBUG) LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT) BUILD_THREADS=$(BUILD_THREADS) V=$(V)
	
#limbo: ndk config qemu
#	$(MAKE) -f $(LIMBO_JNI_ROOT)/limbo.mak all V=1 LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT)

config-debug:
	$(MAKE) createdirs LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT) V=$(V)
	$(MAKE) -f $(LIMBO_JNI_ROOT)/android-qemu-config.mak NDK_DEBUG=1 V=$(V) LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT)
	
config:
	$(MAKE) createdirs LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT)  V=$(V)
	$(MAKE) -f $(LIMBO_JNI_ROOT)/android-qemu-config.mak  V=$(V) LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT)

debug:
	$(MAKE) ndk NDK_DEBUG=1 LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT) V=$(V)
	$(MAKE) qemu NDK_DEBUG=1 LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT) V=$(V)
	$(INSTALL_LIBS)
	$(RENAME_TUNCTL)
	$(RENAME_PARPROUTED)
	$(COPY_ASSETS)
	
release:
	$(MAKE) ndk LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT)  V=$(V)
	$(MAKE) qemu LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT)  V=$(V)
	$(INSTALL_LIBS)
	$(RENAME_TUNCTL)
	$(RENAME_PARPROUTED)
	$(STRIP_LIBS)
	$(REMOVE_GDBSETUP)
	$(REMOVE_GDBSERVER)
	$(COPY_ASSETS)

sync-libs-x86:
	$(SYNC_LIBS)
	$(SYNC_LIBS_X86)

sync-libs-arm:
	$(SYNC_LIBS)
	$(SYNC_LIBS_ARM)

sync-libs-sparc:
	$(SYNC_LIBS)
	$(SYNC_LIBS_SPARC)

sync-libs-ppc:
	$(SYNC_LIBS)
	$(SYNC_LIBS_PPC)

ndk-gdb:
	-echo $(APP_ABI)
	$(COPY_GDB_SETUP_LIBS)
	$(CREATE_OBJDIR_ARMEABI)
	$(CREATE_LIBSDIR_ARMEABI)
	$(COPY_GDB_SETUP_LIBS_ARMEABI)
	$(COPY_GDB_SETUP_OBJ_ARMEABI)
	$(APPEND_GDB_CMD_EXTRA)
	$(ndkgdb) --attach $(PKG_NAME) -v
	
distclean: 
	cd $(LIMBO_JNI_ROOT)/qemu && $(MAKE) distclean LIMBO_JNI_ROOT=$(LIMBO_JNI_ROOT)

clean: distclean ndkclean
	-find . -name "*.d" -exec rm -rf {} \;
	-find . -name "*.a" -exec rm -rf {} \;
	-find . -name "*.o" -exec rm -rf {} \;
	-find . -name "*.so" -exec rm -rf {} \;
	