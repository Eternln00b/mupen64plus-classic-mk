#/* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
# *   Mupen64plus - Makefile                                                *
# *   Mupen64Plus homepage: https://mupen64plus.org/                        *
# *   Copyright (C) 2008-2009 Richard Goedeken                              *
# *   Copyright (C) 2007-2008 DarkJeztr Tillin9                             *
# *                                                                         *
# *   This program is free software; you can redistribute it and/or modify  *
# *   it under the terms of the GNU General Public License as published by  *
# *   the Free Software Foundation; either version 2 of the License, or     *
# *   (at your option) any later version.                                   *
# *                                                                         *
# *   This program is distributed in the hope that it will be useful,       *
# *   but WITHOUT ANY WARRANTY; without even the implied warranty of        *
# *   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the         *
# *   GNU General Public License for more details.                          *
# *                                                                         *
# *   You should have received a copy of the GNU General Public License     *
# *   along with this program; if not, write to the                         *
# *   Free Software Foundation, Inc.,                                       *
# *   51 Franklin Street, Fifth Floor, Boston, MA 02110-1301, USA.          *
# * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * */
# Makefile for Mupen64Plus Core - ARM Cortex-A7 + N00b-Power SDK

# ----------------------------------------------------------------------
# N00b-Power SDK configuration
# ----------------------------------------------------------------------
SDK_ROOT    := /home/maximilien/projects/N00b-Power.SDK
SDK_SYSROOT := $(SDK_ROOT)/sysroot
LINARO_CC   := $(SDK_ROOT)/gcc-linaro/bin

CROSS_COMPILE ?= $(LINARO_CC)/arm-linux-gnueabihf-

SYSROOT_FLAGS := --sysroot=$(SDK_SYSROOT)

PKG_CONFIG_PATH := $(SDK_SYSROOT)/usr/lib/pkgconfig:$(SDK_SYSROOT)/usr/share/pkgconfig
PKG_CONFIG_LIBDIR := $(PKG_CONFIG_PATH)
PKG_CONFIG_SYSROOT_DIR := $(SDK_SYSROOT)

export PKG_CONFIG_PATH
export PKG_CONFIG_LIBDIR
export PKG_CONFIG_SYSROOT_DIR

# ----------------------------------------------------------------------
# Target : ARM 32 bits Cortex-A7 uniquement
# ----------------------------------------------------------------------
CPU := ARM
NEW_DYNAREC := 1
PIC ?= 1

# directory paths
SRCDIR = ../../src
OBJDIR = _obj$(POSTFIX)
SUBDIR = ../../subprojects

# base CFLAGS, LDLIBS, and LDFLAGS
OPTFLAGS ?= -flto -O2 -ffast-math -fno-stack-protector -Wno-unused-function -Wno-deprecated-declarations -Wno-overlength-strings
WARNFLAGS ?= -Wall
CFLAGS += -fvisibility=hidden -I$(SRCDIR) -I$(SRCDIR)/asm_defines -DM64P_PARALLEL
CFLAGS += --sysroot=$(SDK_SYSROOT) -marm -mtune=cortex-a7 -mfpu=neon-vfpv4 -mfloat-abi=hard -I$(SDK_SYSROOT)/usr/include 

CXXFLAGS += -fvisibility-inlines-hidden
LDLIBS += -lm

ifeq ($(PIC), 1)
  CFLAGS += -fPIC
else
  CFLAGS += -fno-PIC
endif

# Linux shared library
TARGET = libmupen64plus$(POSTFIX).so.2.0.0
SONAME = libmupen64plus$(POSTFIX).so.2
LDFLAGS += $(SYSROOT_FLAGS) -Wl,-Bsymbolic -shared -Wl,-export-dynamic -Wl,-soname,$(SONAME)
LDFLAGS += -Wl,-version-script,$(SRCDIR)/api/api_export.ver
LDLIBS += -ldl
LDLIBS += --sysroot=$(SDK_SYSROOT) -L$(SDK_SYSROOT)/usr/lib -L$(SDK_SYSROOT)/lib

ASFLAGS = -f elf

ifeq ($(PIC), 1)
  ASFLAGS += -d PIC
endif

ASFLAGS += -I$(SRCDIR) -I$(SRCDIR)/asm_defines/

# disable verbose output
ifneq ($(findstring $(MAKEFLAGS),s),s)
ifndef V
    Q_CC  = @echo '    CC  '$@;
    Q_CXX = @echo '    CXX '$@;
    Q_AS  = @echo '    AS  '$@;
    Q_LD  = @echo '    LD  '$@;
endif
endif

# test for essential build dependencies
ifeq ($(origin PKG_CONFIG), undefined)
  PKG_CONFIG = pkg-config
  ifeq ($(shell which $(PKG_CONFIG) 2>/dev/null),)
    $(error $(PKG_CONFIG) not found)
  endif
endif

ifeq ($(origin ZLIB_CFLAGS) $(origin ZLIB_LDLIBS), undefined undefined)
  ifeq ($(shell $(PKG_CONFIG) --modversion zlib 2>/dev/null),)
    $(error No zlib development libraries found!)
  endif
  ZLIB_CFLAGS += $(shell $(PKG_CONFIG) --cflags zlib)
  ZLIB_LDLIBS +=  $(shell $(PKG_CONFIG) --libs zlib)
else
  ZLIB_CFLAGS += -I$(SDK_SYSROOT)/usr/include
  ZLIB_LDLIBS += -L$(SDK_SYSROOT)/usr/lib -lz
endif
CFLAGS += $(ZLIB_CFLAGS)
LDLIBS += $(ZLIB_LDLIBS)

ifeq ($(origin MINIZIP_CFLAGS) $(origin MINIZIP_LDLIBS), undefined undefined)
  ifeq ($(shell $(PKG_CONFIG) --modversion minizip 2>/dev/null),)
    CFLAGS += -DNOCRYPT -DNOUNCRYPT -I$(SUBDIR)
    MINIZIP_SOURCE = \
      $(SUBDIR)/minizip/ioapi.c \
      $(SUBDIR)/minizip/zip.c \
      $(SUBDIR)/minizip/unzip.c
  else
    MINIZIP_CFLAGS = $(shell $(PKG_CONFIG) --cflags minizip)
    MINIZIP_LDLIBS =  $(shell $(PKG_CONFIG) --libs minizip)
  endif
endif
CFLAGS += $(MINIZIP_CFLAGS)
LDLIBS += $(MINIZIP_LDLIBS)

ifeq ($(origin LIBPNG_CFLAGS) $(origin LIBPNG_LDLIBS), undefined undefined)
  ifeq ($(shell $(PKG_CONFIG) --modversion libpng 2>/dev/null),)
    $(error No libpng development libraries found!)
  endif
  LIBPNG_CFLAGS += $(shell $(PKG_CONFIG) --cflags libpng)
  LIBPNG_LDLIBS +=  $(shell $(PKG_CONFIG) --libs libpng)
else
  LIBPNG_CFLAGS += -I$(SDK_SYSROOT)/usr/include/libpng16
  LIBPNG_LDLIBS += -L$(SDK_SYSROOT)/usr/lib -lpng -lz
endif
CFLAGS += $(LIBPNG_CFLAGS)
LDLIBS += $(LIBPNG_LDLIBS)

# test for presence of SDL
ifeq ($(origin SDL_CFLAGS) $(origin SDL_LDLIBS), undefined undefined)
  ifeq ($(shell $(PKG_CONFIG) --modversion sdl2 2>/dev/null),)
    $(error No SDL2 development libraries found!)
  endif
  ifeq ($(NETPLAY), 1)
    ifeq ($(shell $(PKG_CONFIG) --modversion SDL2_net 2>/dev/null),)
      $(error No SDL2_net development libraries found!)
    endif
    SDL_CFLAGS += $(shell $(PKG_CONFIG) --cflags SDL2_net)
    SDL_LDLIBS += $(shell $(PKG_CONFIG) --libs SDL2_net)
  endif
  SDL_CFLAGS += $(shell $(PKG_CONFIG) --cflags sdl2)
  SDL_LDLIBS += $(shell $(PKG_CONFIG) --libs sdl2)
else
  SDL_CFLAGS += -I$(SDK_SYSROOT)/usr/include/SDL2 -D_REENTRANT
  SDL_LDLIBS += -L$(SDK_SYSROOT)/usr/lib -lSDL2
endif
CFLAGS += $(SDL_CFLAGS)
LDLIBS += $(SDL_LDLIBS)

# GLES par défaut pour N00b-Power SDK
USE_GLES ?= 1

ifeq ($(USE_GLES), 1)
  GLES_LIB ?= -lGLESv2
  CFLAGS += -DUSE_GLES -DMALI
  LDLIBS += $(GLES_LIB)
  # OSD uses non-ES code and breaks attribs of video plugins
  OSD := 0
endif

OSD ?= 1
ifeq ($(OSD), 1)
  CFLAGS += -DM64P_OSD

  ifeq ($(origin FREETYPE2_CFLAGS) $(origin FREETYPE2_LDLIBS), undefined undefined)
    ifeq ($(shell $(PKG_CONFIG) --modversion freetype2 2>/dev/null),)
      $(error No freetype2 development libraries found!)
    endif
    FREETYPE2_CFLAGS += $(shell $(PKG_CONFIG) --cflags freetype2)
    FREETYPE2_LDLIBS +=  $(shell $(PKG_CONFIG) --libs freetype2)
  endif
  CFLAGS += $(FREETYPE2_CFLAGS)
  LDLIBS += $(FREETYPE2_LDLIBS)

  ifeq ($(origin GL_CFLAGS) $(origin GL_LDLIBS), undefined undefined)
    ifeq ($(shell $(PKG_CONFIG) --modversion gl 2>/dev/null),)
      $(error No OpenGL development libraries found!)
    endif
    GL_CFLAGS += $(shell $(PKG_CONFIG) --cflags gl)
    GL_LDLIBS +=  $(shell $(PKG_CONFIG) --libs gl)
  endif
  CFLAGS += $(GL_CFLAGS)
  LDLIBS += $(GL_LDLIBS)

  ifeq ($(origin GLU_CFLAGS) $(origin GLU_LDLIBS), undefined undefined)
    ifeq ($(shell $(PKG_CONFIG) --modversion glu 2>/dev/null),)
      $(error No OpenGL utility development libraries found!)
    endif
    GLU_CFLAGS += $(shell $(PKG_CONFIG) --cflags glu)
    GLU_LDLIBS +=  $(shell $(PKG_CONFIG) --libs glu)
  endif
  CFLAGS += $(GLU_CFLAGS)
  LDLIBS += $(GLU_LDLIBS)
endif

# set base program pointers and flags
CC       = $(CROSS_COMPILE)gcc
CXX      = $(CROSS_COMPILE)g++
STRINGS  = $(CROSS_COMPILE)strings
AR       = $(CROSS_COMPILE)ar
RANLIB   = $(CROSS_COMPILE)ranlib
AS       = $(CROSS_COMPILE)as
RM       ?= rm -f
INSTALL  ?= install
MKDIR    ?= mkdir -p
COMPILE.c = $(Q_CC)$(CC) $(SYSROOT_FLAGS) $(OPTFLAGS) $(WARNFLAGS) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
COMPILE.cc = $(Q_CXX)$(CXX) $(SYSROOT_FLAGS) $(OPTFLAGS) $(WARNFLAGS) $(CXXFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
COMPILE.as = $(Q_AS)$(AS) $(ASFLAGS)
LINK.o = $(Q_LD)$(CXX) $(SYSROOT_FLAGS) $(OPTFLAGS) $(WARNFLAGS) $(CXXFLAGS) $(LDFLAGS) $(TARGET_ARCH)

LDCONFIG ?= PATH="$$PATH:/sbin" ldconfig -n

# compiler/linker flags for various compile-time options.
ifeq ($(PROFILE), 1)
  CFLAGS += -pg -g
  INSTALL_STRIP_FLAG ?= 
else
  ifeq ($(DEBUG), 1)
    CFLAGS += -g
    OPTFLAGS = -Og
    INSTALL_STRIP_FLAG ?= 
  else
    CFLAGS += -DNDEBUG
    INSTALL_STRIP_FLAG ?= --strip-program=$(CROSS_COMPILE)strip -s
  endif
endif

ifeq ($(LIRC), 1)
  CFLAGS += -DWITH_LIRC
endif
ifeq ($(DEBUGGER), 1)
  CFLAGS += -DDBG
endif
ifeq ($(DBG_COMPARE), 1)
  CFLAGS += -DCOMPARE_CORE
endif
ifeq ($(DBG_CORE), 1)
  CFLAGS += -DCORE_DBG
endif
ifeq ($(DBG_TIMING), 1)
  CFLAGS += -DPROFILE
  LDFLAGS += -lrt
endif

ifneq ($(SHAREDIR),)
  CFLAGS += -DSHAREDIR="$(SHAREDIR)"
endif

# set installation options
ifeq ($(PREFIX),)
  PREFIX := ${SDK_SYSROOT}/usr
endif
ifeq ($(SHAREDIR),)
  SHAREDIR := $(PREFIX)/share/mupen64plus
endif
ifeq ($(LIBDIR),)
  LIBDIR := $(PREFIX)/lib
endif
ifeq ($(INCDIR),)
  INCDIR := $(PREFIX)/include/mupen64plus
endif

# list of required source files for compilation
SOURCE = \
    $(SRCDIR)/api/callbacks.c \
    $(SRCDIR)/api/common.c \
    $(SRCDIR)/api/config.c \
    $(SRCDIR)/api/debugger.c \
    $(SRCDIR)/api/frontend.c \
    $(SRCDIR)/api/vidext.c \
    $(SRCDIR)/backends/api/video_capture_backend.c \
    $(SRCDIR)/backends/plugins_compat/audio_plugin_compat.c \
    $(SRCDIR)/backends/plugins_compat/input_plugin_compat.c \
    $(SRCDIR)/backends/clock_ctime_plus_delta.c \
    $(SRCDIR)/backends/dummy_video_capture.c \
    $(SRCDIR)/backends/file_storage.c \
    $(SRCDIR)/device/cart/cart.c \
    $(SRCDIR)/device/cart/af_rtc.c \
    $(SRCDIR)/device/cart/cart_rom.c \
    $(SRCDIR)/device/cart/eeprom.c \
    $(SRCDIR)/device/cart/flashram.c \
    $(SRCDIR)/device/cart/is_viewer.c \
    $(SRCDIR)/device/cart/sram.c \
    $(SRCDIR)/device/controllers/game_controller.c \
    $(SRCDIR)/device/controllers/vru_controller.c \
    $(SRCDIR)/device/controllers/paks/biopak.c \
    $(SRCDIR)/device/controllers/paks/mempak.c \
    $(SRCDIR)/device/controllers/paks/rumblepak.c \
    $(SRCDIR)/device/controllers/paks/transferpak.c \
    $(SRCDIR)/device/dd/dd_controller.c \
    $(SRCDIR)/device/dd/disk.c \
    $(SRCDIR)/device/device.c \
    $(SRCDIR)/device/gb/gb_cart.c \
    $(SRCDIR)/device/gb/mbc3_rtc.c \
    $(SRCDIR)/device/gb/m64282fp.c \
    $(SRCDIR)/device/memory/memory.c \
    $(SRCDIR)/device/pif/bootrom_hle.c \
    $(SRCDIR)/device/pif/cic.c \
    $(SRCDIR)/device/pif/n64_cic_nus_6105.c \
    $(SRCDIR)/device/pif/pif.c \
    $(SRCDIR)/device/r4300/cached_interp.c \
    $(SRCDIR)/device/r4300/cp0.c \
    $(SRCDIR)/device/r4300/cp1.c \
    $(SRCDIR)/device/r4300/cp2.c \
    $(SRCDIR)/device/r4300/idec.c \
    $(SRCDIR)/device/r4300/interrupt.c \
    $(SRCDIR)/device/r4300/pure_interp.c \
    $(SRCDIR)/device/r4300/r4300_core.c \
    $(SRCDIR)/device/r4300/tlb.c \
    $(SRCDIR)/device/rcp/ai/ai_controller.c \
    $(SRCDIR)/device/rcp/mi/mi_controller.c \
    $(SRCDIR)/device/rcp/pi/pi_controller.c \
    $(SRCDIR)/device/rcp/rdp/fb.c \
    $(SRCDIR)/device/rcp/rdp/rdp_core.c \
    $(SRCDIR)/device/rcp/ri/ri_controller.c \
    $(SRCDIR)/device/rcp/rsp/rsp_core.c \
    $(SRCDIR)/device/rcp/si/si_controller.c \
    $(SRCDIR)/device/rcp/vi/vi_controller.c \
    $(SRCDIR)/device/rdram/rdram.c \
    $(SRCDIR)/main/main.c \
    $(SRCDIR)/main/util.c \
    $(SRCDIR)/main/cheat.c \
    $(SRCDIR)/main/eventloop.c \
    $(SRCDIR)/main/rom.c \
    $(SRCDIR)/main/savestates.c \
    $(SRCDIR)/main/screenshot.c \
    $(SRCDIR)/main/sdl_key_converter.c \
    $(SRCDIR)/main/workqueue.c \
    $(SRCDIR)/plugin/plugin.c \
    $(SRCDIR)/plugin/dummy_video.c \
    $(SRCDIR)/plugin/dummy_audio.c \
    $(SRCDIR)/plugin/dummy_input.c \
    $(SRCDIR)/plugin/dummy_rsp.c \
    $(MINIZIP_SOURCE)

# MD5 lib
SOURCE += \
    $(SUBDIR)/md5/md5.c
CFLAGS += -I$(SUBDIR)/md5

# xxhash
CFLAGS += -I$(SUBDIR)/xxhash

# OSAL (unix only)
SOURCE += \
    $(SRCDIR)/osal/dynamiclib_unix.c \
    $(SRCDIR)/osal/files_unix.c

ifeq ($(OSD), 1)
SOURCE += \
    $(SRCDIR)/osd/osd.c \
    $(SRCDIR)/osd/oglft_c.cpp

SOURCE += $(SUBDIR)/oglft/OGLFT.cpp
CXXFLAGS += -I$(SUBDIR)/oglft
endif

ifeq ($(NETPLAY), 1)
CFLAGS += -DM64P_NETPLAY
SOURCE += $(SRCDIR)/main/netplay.c
endif

ifeq ($(DBG_COUNT), 1)
  CFLAGS += -DCOUNT_INSTR
  SOURCE += $(SRCDIR)/device/r4300/instr_counters.c
endif
ifeq ($(DBG_PROFILE), 1)
  CFLAGS += -DPROFILE_R4300
  SOURCE += $(SRCDIR)/main/profile.c
endif

# dynamic recompiler (ARM 32-bit only)
DYNAREC = arm

ifneq ($(DYNAREC), )
  CFLAGS += -DDYNAREC

  ifeq ($(NEW_DYNAREC), 1)
    CFLAGS += -DNEW_DYNAREC=3
    SOURCE += \
     $(SRCDIR)/device/r4300/new_dynarec/arm/linkage_arm.S \
     $(SRCDIR)/device/r4300/new_dynarec/arm/arm_cpu_features.c

    SOURCE += \
      $(SRCDIR)/device/r4300/new_dynarec/new_dynarec.c
  else
    SOURCE += \
      $(SRCDIR)/device/r4300/recomp.c \
      $(SRCDIR)/device/r4300/$(DYNAREC)/assemble.c \
      $(SRCDIR)/device/r4300/$(DYNAREC)/dynarec.c \
      $(SRCDIR)/device/r4300/$(DYNAREC)/regcache.c \
      $(SRCDIR)/device/r4300/$(DYNAREC)/dyna_start.asm
  endif
endif

ifeq ($(LIRC), 1)
  SOURCE += $(SRCDIR)/main/lirc.c
  LDLIBS += -llirc_client
endif

ifeq ($(KEYBINDINGS), 0)
	CFLAGS += -DNO_KEYBINDINGS
endif

ifeq ($(ACCURATE_FPU), 1)
  CFLAGS += -DACCURATE_FPU_BEHAVIOR
endif

ifeq ($(DEBUGGER), 1)
  SOURCE += \
    $(SRCDIR)/debugger/dbg_debugger.c \
    $(SRCDIR)/debugger/dbg_decoder.c \
    $(SRCDIR)/debugger/dbg_memory.c \
    $(SRCDIR)/debugger/dbg_breakpoints.c

  DEBUGGER_NO_DISASM ?= 0

  ifeq ($(DEBUGGER_NO_DISASM),1)
    CFLAGS += -DNO_HOST_DISASSEMBLER
  else
    LDLIBS += -lopcodes -lbfd

    LIBOPCODES_VERSION := $(shell $(STRINGS) --version | head -n1 | rev | cut -d ' ' -f1 | rev)
    LIBOPCODES_MAJOR := $(shell echo $(LIBOPCODES_VERSION) | cut -f1 -d.)
    LIBOPCODES_MINOR := $(shell echo $(LIBOPCODES_VERSION) | cut -f2 -d.)
    LIBOPCODES_POINT := $(shell echo $(LIBOPCODES_VERSION) | cut -f3 -d. | sed 's/^$$/0/')
    LIBOPCODES_GE_2_29 := $(shell [ $(LIBOPCODES_MAJOR) -gt 2 -o \( $(LIBOPCODES_MAJOR) -eq 2 -a $(LIBOPCODES_MINOR) -ge 28 \) -o \( $(LIBOPCODES_MAJOR) -eq 2 -a $(LIBOPCODES_MINOR) -eq 28 -a $(LIBOPCODES_POINT) -ge 1 \) ] && echo true)
    LIBBFD_GE_2_39 := $(shell [ $(LIBOPCODES_MAJOR) -gt 2 -o \( $(LIBOPCODES_MAJOR) -eq 2 -a $(LIBOPCODES_MINOR) -ge 39 \) -o \( $(LIBOPCODES_MAJOR) -eq 2 -a $(LIBOPCODES_MINOR) -eq 39 -a $(LIBOPCODES_POINT) -ge 1 \) ] && echo true)
    ifeq ($(LIBOPCODES_GE_2_29),true)
      CFLAGS += -DUSE_LIBOPCODES_GE_2_29
    endif
    ifeq ($(LIBBFD_GE_2_39),true)
      CFLAGS += -DUSE_LIBBFD_GE_2_39
    endif
  endif
endif

ifeq ($(OPENCV), 1)
  SOURCE += $(SRCDIR)/backends/opencv_video_capture.cpp
  CFLAGS += -DM64P_OPENCV
endif

# generate a list of object files to build, make a temporary directory for them
OBJECTS := $(patsubst $(SRCDIR)/%.c,   $(OBJDIR)/%.o, $(filter $(SRCDIR)/%.c,   $(SOURCE)))
OBJECTS += $(patsubst $(SUBDIR)/%.c,   $(OBJDIR)/subprojects/%.o, $(filter $(SUBDIR)/%.c,   $(SOURCE)))
OBJECTS += $(patsubst $(SRCDIR)/%.c,   $(OBJDIR)/%.o, $(SRCDIR)/asm_defines/asm_defines.c)
OBJECTS += $(patsubst $(SRCDIR)/%.cpp, $(OBJDIR)/%.o, $(filter $(SRCDIR)/%.cpp, $(SOURCE)))
OBJECTS += $(patsubst $(SUBDIR)/%.cpp, $(OBJDIR)/subprojects/%.o, $(filter $(SUBDIR)/%.cpp, $(SOURCE)))
OBJECTS += $(patsubst $(SRCDIR)/%.S, $(OBJDIR)/%.o, $(filter %.S, $(SOURCE)))
OBJECTS += $(patsubst $(SRCDIR)/%.asm, $(OBJDIR)/%.o, $(filter %.asm, $(SOURCE)))
OBJDIRS = $(dir $(OBJECTS))
$(shell $(MKDIR) $(OBJDIRS))

# we have a special object file for the new dynarec
ASM_DEFINES_OBJ = $(OBJDIR)/asm_defines/asm_defines.o

# build targets
targets:
	@echo "Mupen64Plus-core makefile (ARM Cortex-A7 + N00b-Power SDK)."
	@echo "  Targets:"
	@echo "    all            == Build Mupen64Plus core library"
	@echo "    clean          == remove object files"
	@echo "    install        == Install Mupen64Plus core library"
	@echo "    uninstall      == Uninstall Mupen64Plus core library"
	@echo "  Build Options:"
	@echo "    LIRC=1         == enable LIRC support"
	@echo "    NO_ASM=1       == build without assembly (no dynamic recompiler)"
	@echo "    USE_GLES=1     == build against GLESv2 instead of OpenGL (default: 1)"
	@echo "    VC=1           == build against Broadcom Videocore GLESv2"
	@echo "    SHAREDIR=path  == extra path to search for shared data files"
	@echo "    OPTFLAGS=flag  == compiler optimization (default: -O3 -flto)"
	@echo "    WARNFLAGS=flag == compiler warning levels (default: -Wall)"
	@echo "    PIC=(1|0)      == Force enable/disable of position independent code"
	@echo "    OSD=(1|0)      == Enable/disable build of OpenGL On-screen display"
	@echo "    NETPLAY=1      == Enable netplay functionality, requires SDL2_net"
	@echo "    NEW_DYNAREC=1  == Replace dynamic recompiler with Ari64's experimental dynarec"
	@echo "    KEYBINDINGS=0  == Disables the default keybindings"
	@echo "    ACCURATE_FPU=1 == Enables accurate FPU behavior (i.e correct cause bits)"
	@echo "    OPENCV=1       == Enable OpenCV support"
	@echo "    POSTFIX=name   == String added to the name of the the build (default: '')"
	@echo "  Install Options:"
	@echo "    PREFIX=path    == install/uninstall prefix (default: /usr/local)"
	@echo "    SHAREDIR=path  == path to install shared data files (default: PREFIX/share/mupen64plus)"
	@echo "    LIBDIR=path    == path to install core library (default: PREFIX/lib)"
	@echo "    INCDIR=path    == path to install core header files (default: PREFIX/include/mupen64plus)"
	@echo "    DESTDIR=path   == path to prepend to all installation paths (only for packagers)"
	@echo "  Debugging Options:"
	@echo "    PROFILE=1      == build gprof instrumentation into binaries for profiling"
	@echo "    DEBUG=1        == add debugging symbols to binaries"
	@echo "    DEBUGGER=1     == build debugger API into core for front-ends.  runs slower."
	@echo "    DBG_CORE=1     == print debugging info in r4300 core"
	@echo "    DBG_COMPARE=1  == enable core-synchronized r4300 debugging"
	@echo "    DBG_TIMING=1   == print timing data"
	@echo "    DBG_PROFILE=1  == dump profiling data for r4300 dynarec to data file"
	@echo "    V=1            == show verbose compiler output"

all: $(TARGET)

install: $(TARGET)
	$(INSTALL) -d "$(DESTDIR)$(LIBDIR)"
	$(INSTALL) -m 0644 $(INSTALL_STRIP_FLAG) $(TARGET) "$(DESTDIR)$(LIBDIR)"
	$(INSTALL) -d "$(DESTDIR)$(SHAREDIR)"
	$(INSTALL) -m 0644 $(SRCDIR)/../data/* "$(DESTDIR)$(SHAREDIR)"
	$(INSTALL) -d "$(DESTDIR)$(INCDIR)"
	$(INSTALL) -m 0644 $(SRCDIR)/api/m64p_*.h "$(DESTDIR)$(INCDIR)"
	-$(LDCONFIG) "$(DESTDIR)$(LIBDIR)"
	if [ ! -e "$(DESTDIR)$(LIBDIR)/$(SONAME)" ]; then ln -sf "$(TARGET)" "$(DESTDIR)$(LIBDIR)/$(SONAME)"; fi

uninstall:
	$(RM) "$(DESTDIR)$(LIBDIR)/$(TARGET)"
	if [ "$(SONAME)" != "" ]; then $(RM) "$(DESTDIR)$(LIBDIR)/$(SONAME)"; fi
	$(RM) $(DESTDIR)$(INCDIR)/m64p_*.h
	$(RM) "$(DESTDIR)$(SHAREDIR)/mupen64plus.ini"
	$(RM) "$(DESTDIR)$(SHAREDIR)/font.ttf"
	$(RM) "$(DESTDIR)$(SHAREDIR)/mupencheat.txt"

clean:
	$(RM) -r _obj $(OBJDIR) $(TARGET) $(SONAME) $(SRCDIR)/asm_defines/asm_defines_*.h

# build dependency files
CFLAGS += -MD -MP
-include $(OBJECTS:.o=.d)

CXXFLAGS += $(CFLAGS)

# It is important to disable LTO for this object file
# otherwise we can't extract usefull information from it.
$(ASM_DEFINES_OBJ): $(SRCDIR)/asm_defines/asm_defines.c
	$(COMPILE.c) -fno-lto -o $@ $<

# Script hackery for generating ASM include files for the new dynarec assembly code
$(SRCDIR)/asm_defines/asm_defines_gas.h: $(SRCDIR)/asm_defines/asm_defines_nasm.h
$(SRCDIR)/asm_defines/asm_defines_nasm.h: $(ASM_DEFINES_OBJ) $(SRCDIR)/../tools/gen_asm_script.sh
	bash $(SRCDIR)/../tools/gen_asm_script.sh "$(SRCDIR)/asm_defines" "$(ASM_DEFINES_OBJ)"

# standard build rules
$(OBJDIR)/%.o: $(SRCDIR)/%.asm $(SRCDIR)/asm_defines/asm_defines_nasm.h
	$(COMPILE.as) -o $@ $<

$(OBJDIR)/%.o: $(SRCDIR)/%.S $(SRCDIR)/asm_defines/asm_defines_gas.h
	$(COMPILE.c) -o $@ $<

$(OBJDIR)/%.o: $(SRCDIR)/%.c
	$(COMPILE.c) -o $@ $<

$(OBJDIR)/%.o: $(SRCDIR)/%.cpp
	$(COMPILE.cc) -o $@ $<

$(OBJDIR)/subprojects/%.o: $(SUBDIR)/%.c
	$(COMPILE.c) -o $@ $<

$(OBJDIR)/subprojects/%.o: $(SUBDIR)/%.cpp
	$(COMPILE.cc) -o $@ $<

$(TARGET): $(OBJECTS)
	$(LINK.o) $^ $(LOADLIBES) $(LDLIBS) -o $@
	if [ "$(SONAME)" != "" ]; then ln -sf $@ $(SONAME); fi

.PHONY: all clean install uninstall targets