#/* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
# *   Mupen64plus-input-sdl - Makefile (N00b-Power SDK / ARMv7)             *
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
# pkg-config cross-compilé : pointer vers le sysroot du SDK
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
SO_EXTENSION := so
SHARED := -shared

HOST_CPU := armv7
CPU := ARM
ARCH_DETECTED := 32BITS
PIC := 1

SRCDIR = ../../src
OBJDIR = _obj$(POSTFIX)

# base CFLAGS, LDLIBS, and LDFLAGS
OPTFLAGS ?= -flto -O2 -fno-stack-protector -Wno-unused-function -Wno-deprecated-declarations -Wno-overlength-strings
WARNFLAGS ?= -Wall
CFLAGS += $(OPTFLAGS) $(WARNFLAGS) $(ARCH_FLAGS) \
          -ffast-math -fvisibility=hidden \
          -I$(SRCDIR) -D_GNU_SOURCE=1 -I$(SDK_SYSROOT)/usr/include
LDFLAGS += $(SYSROOT_FLAGS) $(SHARED)
LDLIBS += -lm

# PIC obligatoire pour un .so ARM
CFLAGS += -fPIC
LDFLAGS += -fPIC

# flags Linux
LDLIBS += -ldl

# ----------------------------------------------------------------------
# Détection de pkg-config + SDL2
# ----------------------------------------------------------------------
PKG_CONFIG ?= $(CROSS_COMPILE)pkg-config

ifeq ($(origin PKG_CONFIG), undefined)
  ifeq ($(shell which $(PKG_CONFIG) 2>/dev/null),)
    $(error $(PKG_CONFIG) not found)
  endif
endif

SDL_CFLAGS ?= -I$(SDK_SYSROOT)/usr/include/SDL2 -D_REENTRANT
SDL_LDLIBS ?= -L$(SDK_SYSROOT)/usr/lib -lSDL2

CFLAGS += $(SDL_CFLAGS)
LDLIBS += $(SDL_LDLIBS)


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
    Q_CC = @echo '    CC  '$@;
    Q_LD = @echo '    LD  '$@;
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
# Debug / strip
# ----------------------------------------------------------------------
ifeq ($(DEBUG),1)
  CFLAGS += -g
  INSTALL_STRIP_FLAG ?=
else
  CFLAGS += -DNDEBUG
  INSTALL_STRIP_FLAG ?= --strip-program=$(CROSS_COMPILE)strip -s
endif

ifeq ($(PLUGINDBG), 1)
  CFLAGS += -D_DEBUG
endif

# ----------------------------------------------------------------------
# Installation options (dans le sysroot par défaut)
# ----------------------------------------------------------------------
PREFIX     ?= $(SDK_SYSROOT)/usr
SHAREDIR   ?= $(PREFIX)/share/mupen64plus
LIBDIR     ?= $(PREFIX)/lib
PLUGINDIR  ?= $(LIBDIR)/mupen64plus

# ----------------------------------------------------------------------
# Sources (Unix uniquement, pas de osal_win32)
# ----------------------------------------------------------------------
SOURCE = \
	$(SRCDIR)/plugin.c \
	$(SRCDIR)/autoconfig.c \
	$(SRCDIR)/sdl_key_converter.c \
	$(SRCDIR)/config.c \
	$(SRCDIR)/osal_dynamiclib_unix.c

# ----------------------------------------------------------------------
# Génération de la liste des objets + création des dossiers
# ----------------------------------------------------------------------
OBJECTS := $(patsubst $(SRCDIR)/%.c, $(OBJDIR)/%.o, $(filter %.c, $(SOURCE)))
OBJDIRS = $(dir $(OBJECTS))
$(shell $(MKDIR) $(OBJDIRS))

# ----------------------------------------------------------------------
# Cible
# ----------------------------------------------------------------------
TARGET = mupen64plus-input-sdl$(POSTFIX).$(SO_EXTENSION)

targets:
	@echo "Mupen64Plus-input-sdl makefile (Linux ARMv7 + N00b-Power SDK)."
	@echo "  Targets:"
	@echo "    all           == Build Mupen64Plus SDL input plugin"
	@echo "    clean         == remove object files"
	@echo "    rebuild       == clean and re-build all"
	@echo "    install       == Install Mupen64Plus SDL input plugin"
	@echo "    uninstall     == Uninstall Mupen64Plus SDL input plugin"
	@echo "  Options:"
	@echo "    APIDIR=path   == path to find Mupen64Plus Core headers"
	@echo "    OPTFLAGS=flag == compiler optimization (default: -O3 -flto)"
	@echo "    WARNFLAGS=flag == compiler warning levels (default: -Wall)"
	@echo "    PIC=(1|0)     == Force enable/disable of position independent code"
	@echo "    POSTFIX=name  == String added to the name of the build (default: '')"
	@echo "  Install Options:"
	@echo "    PREFIX=path   == install/uninstall prefix (default: SDK sysroot)"
	@echo "    SHAREDIR=path == path to install shared data files"
	@echo "    LIBDIR=path   == library prefix"
	@echo "    PLUGINDIR=path == path to install plugin libraries"
	@echo "    DESTDIR=path  == path to prepend to all installation paths"
	@echo "  Debugging Options:"
	@echo "    DEBUG=1       == add debugging symbols"
	@echo "    PLUGINDBG=1   == print extra debugging information while running"
	@echo "    V=1           == show verbose compiler output"

all: $(TARGET)

install: $(TARGET)
	$(INSTALL) -d "$(DESTDIR)$(PLUGINDIR)"
	$(INSTALL) -m 0644 $(INSTALL_STRIP_FLAG) $(TARGET) "$(DESTDIR)$(PLUGINDIR)"
	$(INSTALL) -d "$(DESTDIR)$(SHAREDIR)"
	$(INSTALL) -m 0644 "$(SRCDIR)/../data/InputAutoCfg.ini" "$(DESTDIR)$(SHAREDIR)"

uninstall:
	$(RM) "$(DESTDIR)$(PLUGINDIR)/$(TARGET)"
	$(RM) "$(DESTDIR)$(SHAREDIR)/InputAutoCfg.ini"

clean:
	$(RM) -r $(OBJDIR) $(TARGET)

rebuild: clean all

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