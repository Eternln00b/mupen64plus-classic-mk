#/* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
# *   Mupen64plus-ui-console - Makefile (N00b-Power SDK / ARMv7)            *
# *   Adapté pour cross-compilation vers Nintendo Classic Mini              *
# * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * */

# ----------------------------------------------------------------------
# N00b-Power SDK configuration
# ----------------------------------------------------------------------
SDK_ROOT    := /home/maximilien/projects/N00b-Power.SDK
SDK_SYSROOT := $(SDK_ROOT)/sysroot
LINARO_CC   := $(SDK_ROOT)/gcc-linaro/bin

CROSS_COMPILE ?= $(LINARO_CC)/arm-linux-gnueabihf-
CC            := $(CROSS_COMPILE)gcc
CXX           := $(CROSS_COMPILE)g++
ARCH_FLAGS    := -marm -mtune=cortex-a7 -mfpu=neon-vfpv4 -mfloat-abi=hard

# ----------------------------------------------------------------------
# pkg-config : hôte + SYSROOT_DIR (car pas de pkg-config cross dans Linaro)
# ----------------------------------------------------------------------
SYSROOT_FLAGS := --sysroot=$(SDK_SYSROOT)

PKG_CONFIG_PATH        := $(SDK_SYSROOT)/usr/lib/pkgconfig:$(SDK_SYSROOT)/usr/share/pkgconfig
PKG_CONFIG_LIBDIR      := $(PKG_CONFIG_PATH)
PKG_CONFIG_SYSROOT_DIR := $(SDK_SYSROOT)

export PKG_CONFIG_PATH
export PKG_CONFIG_LIBDIR
export PKG_CONFIG_SYSROOT_DIR

# ----------------------------------------------------------------------
# On force Linux / ARMv7 (plus de détection uname)
# ----------------------------------------------------------------------
OS := LINUX
EXEEXT :=

HOST_CPU := armv7
CPU := ARM
ARCH_DETECTED := 32BITS

SRCDIR = ../../src
OBJDIR = _obj$(POSTFIX)

# base CFLAGS, LDLIBS, and LDFLAGS
OPTFLAGS ?= -flto -O2 -fno-stack-protector -Wno-unused-function -Wno-deprecated-declarations
WARNFLAGS ?= -Wall

CFLAGS += $(OPTFLAGS) $(WARNFLAGS) $(ARCH_FLAGS) \
          -ffast-math -I$(SRCDIR) -D_GNU_SOURCE=1 \
          -I$(SDK_SYSROOT)/usr/include 

CFLAGS += -pthread
LDLIBS += -pthread
LDLIBS += -ldl
LDLIBS += -L$(SDK_SYSROOT)/usr/lib -lasound

# ----------------------------------------------------------------------
# PIE : on force -no-pie pour éviter les problèmes avec le toolchain
# ----------------------------------------------------------------------
PIE ?= 0
ifeq ($(PIE), 1)
  CFLAGS += -fPIE
  LDFLAGS += -pie
else
  CFLAGS += -fno-PIE
  LDFLAGS += -no-pie
endif

# ----------------------------------------------------------------------
# pkg-config + SDL2 (fallback manuel si pkg-config échoue)
# ----------------------------------------------------------------------
PKG_CONFIG ?= pkg-config

ifeq ($(origin SDL_CFLAGS) $(origin SDL_LDLIBS), undefined undefined)
  ifneq ($(shell $(PKG_CONFIG) --modversion sdl2 2>/dev/null),)
    SDL_CFLAGS += $(shell $(PKG_CONFIG) --cflags sdl2)
    SDL_LDLIBS += $(shell $(PKG_CONFIG) --libs sdl2)
  else
    SDL_CFLAGS += -I$(SDK_SYSROOT)/usr/include/SDL2 -D_REENTRANT
    SDL_LDLIBS += -L$(SDK_SYSROOT)/usr/lib -lSDL2
  endif
endif
CFLAGS += $(SDL_CFLAGS)
LDLIBS += $(SDL_LDLIBS)

# ----------------------------------------------------------------------
# Endianness (ARMv7 = little-endian, donc pas de BIG_ENDIAN)
# ----------------------------------------------------------------------
# ifeq ($(BIG_ENDIAN), 1)
#   CFLAGS += -DM64P_BIG_ENDIAN
# endif

# ----------------------------------------------------------------------
# Headers Mupen64Plus Core (via votre sysroot)
# ----------------------------------------------------------------------
ifneq ("$(APIDIR)","")
  CFLAGS += "-I$(APIDIR)"
else
  TRYDIR = $(SDK_SYSROOT)/usr/include/mupen64plus
  ifneq ("$(wildcard $(TRYDIR)/m64p_types.h)","")
    CFLAGS += -I$(TRYDIR)
  else
    $(error Mupen64Plus API header files not found! Use APIDIR=path)
  endif
endif

# ----------------------------------------------------------------------
# Output réduit (sauf V=1)
# ----------------------------------------------------------------------
ifneq ($(findstring $(MAKEFLAGS),s),s)
ifndef V
	Q_CC  = @echo '    CC  '$@;
	Q_LD  = @echo '    LD  '$@;
endif
endif

# ----------------------------------------------------------------------
# Base program pointers and flags
# ----------------------------------------------------------------------
RM       ?= rm -f
INSTALL  ?= install
MKDIR    ?= mkdir -p
COMPILE.c = $(Q_CC)$(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
LINK.o = $(Q_LD)$(CC) $(CFLAGS) $(LDFLAGS) $(TARGET_ARCH)

# ----------------------------------------------------------------------
# Installation options (dans le sysroot par défaut)
# ----------------------------------------------------------------------
PREFIX     ?= $(SDK_SYSROOT)/usr
LIBDIR     ?= $(PREFIX)/lib
BINDIR     ?= $(PREFIX)/bin
MANDIR     ?= $(PREFIX)/share/man
APPSDIR    ?= $(PREFIX)/share/applications
ICONSDIR   ?= $(PREFIX)/share/icons/hicolor

# ----------------------------------------------------------------------
# Chemins de plugins/core/shared (compilés dans le binaire)
# ----------------------------------------------------------------------
PLUGINDIR  ?= /usr/lib/mupen64plus/
COREDIR    ?= /usr/lib/
SHAREDIR   ?= /usr/share/mupen64plus/

CFLAGS += '-DPLUGINDIR="$(PLUGINDIR)"'
CFLAGS += '-DCOREDIR="$(COREDIR)"'
CFLAGS += '-DSHAREDIR="$(SHAREDIR)"'

# ----------------------------------------------------------------------
# Debug / strip
# ----------------------------------------------------------------------
ifeq ($(DEBUG),1)
  CFLAGS += -g
  INSTALL_STRIP_FLAG ?=
else
  INSTALL_STRIP_FLAG ?= --strip-program=$(CROSS_COMPILE)strip -s
endif

# ----------------------------------------------------------------------
# Sources (Unix uniquement)
# ----------------------------------------------------------------------
SOURCE = \
	$(SRCDIR)/cheat.c \
	$(SRCDIR)/compare_core.c \
	$(SRCDIR)/core_interface.c \
	$(SRCDIR)/debugger.c \
	$(SRCDIR)/main.c \
	$(SRCDIR)/plugin.c \
	$(SRCDIR)/osal_dynamiclib_unix.c \
	$(SRCDIR)/osal_files_unix.c

# ----------------------------------------------------------------------
# Génération de la liste des objets + création des dossiers
# ----------------------------------------------------------------------
OBJECTS := $(patsubst $(SRCDIR)/%.c, $(OBJDIR)/%.o, $(filter %.c, $(SOURCE)))
OBJDIRS = $(dir $(OBJECTS))
$(shell $(MKDIR) $(OBJDIRS))

# ----------------------------------------------------------------------
# Cible
# ----------------------------------------------------------------------
TARGET = mupen64plus$(POSTFIX)$(EXEEXT)

targets:
	@echo "Mupen64Plus-ui-console makefile (Linux ARMv7 + N00b-Power SDK)."
	@echo "  Targets:"
	@echo "    all            == Build Mupen64Plus console front-end application"
	@echo "    clean          == remove object files and build products"
	@echo "    rebuild        == clean and re-build all"
	@echo "    install        == Install Mupen64Plus console front-end application"
	@echo "    uninstall      == Uninstall Mupen64Plus console front-end application"
	@echo "  Options:"
	@echo "    COREDIR=path   == default path to search for Mupen64Plus Core (must end with slash)"
	@echo "    PLUGINDIR=path == default path to search for plugins"
	@echo "    SHAREDIR=path  == default path to search for shared data files"
	@echo "    APIDIR=path    == path to find Mupen64Plus Core headers"
	@echo "    OPTFLAGS=flags == compiler optimization (default: -O2 -flto)"
	@echo "    WARNFLAGS=flag == compiler warning levels (default: -Wall)"
	@echo "    PIE=(1|0)      == Force enable/disable of position independent executables"
	@echo "    POSTFIX=name   == String added to the name of the build (default: '')"
	@echo "  Install Options:"
	@echo "    PREFIX=path    == install/uninstall prefix (default: SDK sysroot)"
	@echo "    BINDIR=path    == path to install mupen64plus binary"
	@echo "    MANDIR=path    == path to install mupen64plus manual page"
	@echo "    APPSDIR=path   == path to install desktop file"
	@echo "    ICONSDIR=path  == path to install icon files"
	@echo "    DESTDIR=path   == path to prepend to all installation paths"
	@echo "  Debugging Options:"
	@echo "    DEBUG=1        == add debugging symbols to application binary"
	@echo "    V=1            == show verbose compiler output"

all: $(TARGET)

clean:
	$(RM) -r $(OBJDIR) $(TARGET)

rebuild: clean all

install: $(TARGET)
	$(INSTALL) -d "$(DESTDIR)$(BINDIR)"
	$(INSTALL) -m 0755 $(INSTALL_STRIP_FLAG) $(TARGET) "$(DESTDIR)$(BINDIR)"
	$(INSTALL) -d "$(DESTDIR)$(MANDIR)/man6"
	$(INSTALL) -m 0644 $(SRCDIR)/../doc/mupen64plus.6 "$(DESTDIR)$(MANDIR)/man6"
	$(INSTALL) -d "$(DESTDIR)$(APPSDIR)"
	$(INSTALL) -m 0644 $(SRCDIR)/../data/mupen64plus.desktop "$(DESTDIR)$(APPSDIR)"
	$(INSTALL) -d "$(DESTDIR)$(ICONSDIR)/48x48/apps"
	$(INSTALL) -m 0644 $(SRCDIR)/../data/icons/48x48/apps/mupen64plus.png "$(DESTDIR)$(ICONSDIR)/48x48/apps"
	$(INSTALL) -d "$(DESTDIR)$(ICONSDIR)/scalable/apps"
	$(INSTALL) -m 0644 $(SRCDIR)/../data/icons/scalable/apps/mupen64plus.svg "$(DESTDIR)$(ICONSDIR)/scalable/apps"

uninstall:
	$(RM) "$(DESTDIR)$(BINDIR)/$(TARGET)" "$(DESTDIR)$(MANDIR)/man6/mupen64plus.6"
	$(RM) "$(DESTDIR)$(APPSDIR)/mupen64plus.desktop"
	$(RM) "$(DESTDIR)$(ICONSDIR)/48x48/apps/mupen64plus.png"
	$(RM) "$(DESTDIR)$(ICONSDIR)/scalable/apps/mupen64plus.svg"

# ----------------------------------------------------------------------
# Dépendances
# ----------------------------------------------------------------------
CFLAGS += -MD -MP
-include $(OBJECTS:.o=.d)

# ----------------------------------------------------------------------
# Règles standard
# ----------------------------------------------------------------------
$(OBJDIR)/%.o: $(SRCDIR)/%.c
	$(COMPILE.c) -o $@ $<

$(TARGET): $(OBJECTS)
	$(LINK.o) $^ $(LOADLIBES) $(LDLIBS) -o $@

.PHONY: all clean install uninstall targets