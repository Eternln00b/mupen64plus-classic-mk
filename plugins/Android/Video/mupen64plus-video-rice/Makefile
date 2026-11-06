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

SYSROOT_FLAGS := --sysroot=$(SDK_SYSROOT)

PKG_CONFIG_PATH := $(SDK_SYSROOT)/usr/lib/pkgconfig:$(SDK_SYSROOT)/usr/share/pkgconfig
PKG_CONFIG_LIBDIR := $(PKG_CONFIG_PATH)
PKG_CONFIG_SYSROOT_DIR := $(SDK_SYSROOT)

export PKG_CONFIG_PATH
export PKG_CONFIG_LIBDIR
export PKG_CONFIG_SYSROOT_DIR

# ----------------------------------------------------------------------
# Mupen64Plus Video Rice Makefile (Linux ARMv7 + GLES2 + MALI)
# ----------------------------------------------------------------------

# OS : Linux uniquement
OS := LINUX
SO_EXTENSION := so
SHARED := -shared

# Architecture : ARM 32-bit uniquement
HOST_CPU := armv7
CPU := ARM
ARCH_DETECTED := 32BITS
PIC := 1
NO_ASM := 1

# GLES2 (obligatoire pour Mali-400)
USE_GLES := 1

# base CFLAGS, LDLIBS, and LDFLAGS
OPTFLAGS ?= -flto -O2 -fno-stack-protector -Wno-unused-function -Wno-deprecated-declarations -Wno-overlength-strings
WARNFLAGS ?= -Wall
CFLAGS += $(OPTFLAGS) $(WARNFLAGS) $(ARCH_FLAGS) \
          -ffast-math -fno-strict-aliasing -fvisibility=hidden \
          -I../../src -I$(SDK_SYSROOT)/usr/include -DGCC 
		  
CXXFLAGS += -fvisibility-inlines-hidden -std=gnu++11
LDFLAGS += $(SYSROOT_FLAGS) $(SHARED)

# Position independent code
CFLAGS += -fPIC
LDFLAGS += -fPIC -static-libstdc++ -static-libgcc

# set special flags per-system (Linux uniquement)
LDLIBS += -ldl
LDFLAGS += -Wl,-version-script,$(SRCDIR)/video_api_export.ver

# set mupen64plus core API header path
ifneq ("$(APIDIR)","")
  CFLAGS += "-I$(APIDIR)"
else
  TRYDIR = $(SDK_SYSROOT)/usr/include/mupen64plus
  ifneq ("$(wildcard $(TRYDIR)/m64p_types.h)","")
    CFLAGS += -I$(TRYDIR)
  else
    $(error Mupen64Plus API header files not found! Use makefile parameter APIDIR to force a location.)
  endif
endif

# pkg-config pour libpng et SDL2
ifneq ($(shell $(CROSS_COMPILE)pkg-config --modversion libpng 2>/dev/null),)
  LIBPNG_CFLAGS += $(shell $(CROSS_COMPILE)pkg-config --cflags libpng)
  LIBPNG_LDLIBS += $(shell $(CROSS_COMPILE)pkg-config --libs libpng)
else
  LIBPNG_CFLAGS += -I$(SDK_SYSROOT)/usr/include/libpng16
  LIBPNG_LDLIBS += -L$(SDK_SYSROOT)/usr/lib -lpng -lz
endif
CFLAGS += $(LIBPNG_CFLAGS)
LDLIBS += $(LIBPNG_LDLIBS)

ifneq ($(shell $(CROSS_COMPILE)pkg-config --modversion sdl2 2>/dev/null),)
  SDL_CFLAGS += $(shell $(CROSS_COMPILE)pkg-config --cflags sdl2)
  SDL_LDLIBS += $(shell $(CROSS_COMPILE)pkg-config --libs sdl2)
else
  SDL_CFLAGS += -I$(SDK_SYSROOT)/usr/include/SDL2 -D_REENTRANT
  SDL_LDLIBS += -L$(SDK_SYSROOT)/usr/lib -lSDL2
endif
CFLAGS += $(SDL_CFLAGS)
LDLIBS += $(SDL_LDLIBS)

# GLES2 (Mali-400) + libmali
ifeq ($(USE_GLES), 1)
  CFLAGS += -DUSE_GLES -DMALI
  LDLIBS += -lGLESv2
endif

# reduced compile output
ifneq ($(findstring $(MAKEFLAGS),s),s)
ifndef V
	Q_CC  = @echo '    CC  '$@;
	Q_CXX = @echo '    CXX '$@;
	Q_LD  = @echo '    LD  '$@;
endif
endif

# set base program pointers and flags
RM       ?= rm -f
INSTALL  ?= install
MKDIR ?= mkdir -p
COMPILE.c = $(Q_CC)$(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
COMPILE.cc = $(Q_CXX)$(CXX) $(CXXFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
LINK.o = $(Q_LD)$(CXX) $(CXXFLAGS) $(LDFLAGS) $(TARGET_ARCH)

# set special flags for given Makefile parameters
ifeq ($(DEBUG),1)
  CFLAGS += -g
  INSTALL_STRIP_FLAG ?= 
else
  INSTALL_STRIP_FLAG ?= --strip-program=$(CROSS_COMPILE)strip -s
endif
ifeq ($(NO_ASM), 1)
  CFLAGS += -DNO_ASM
endif

# set installation options
PREFIX ?= ${SDK_SYSROOT}/usr
SHAREDIR ?= $(PREFIX)/share/mupen64plus
LIBDIR ?= $(PREFIX)/lib
PLUGINDIR ?= $(LIBDIR)/mupen64plus

SRCDIR = ../../src
OBJDIR = _obj$(POSTFIX)

# list of source files to compile
SOURCE = \
	$(SRCDIR)/liblinux/BMGImage.c \
	$(SRCDIR)/liblinux/BMGUtils.cpp \
	$(SRCDIR)/liblinux/bmp.c \
	$(SRCDIR)/liblinux/pngrw.c \
	$(SRCDIR)/Blender.cpp \
	$(SRCDIR)/Combiner.cpp \
	$(SRCDIR)/Config.cpp \
	$(SRCDIR)/ConvertImage.cpp \
	$(SRCDIR)/ConvertImage16.cpp \
	$(SRCDIR)/Debugger.cpp \
	$(SRCDIR)/DeviceBuilder.cpp \
	$(SRCDIR)/FrameBuffer.cpp \
	$(SRCDIR)/GraphicsContext.cpp \
	$(SRCDIR)/OGLCombiner.cpp \
	$(SRCDIR)/OGLExtensions.cpp \
	$(SRCDIR)/OGLGraphicsContext.cpp \
	$(SRCDIR)/OGLRender.cpp \
	$(SRCDIR)/OGLRenderExt.cpp \
	$(SRCDIR)/OGLTexture.cpp \
	$(SRCDIR)/Render.cpp \
	$(SRCDIR)/RenderBase.cpp \
	$(SRCDIR)/RenderExt.cpp \
	$(SRCDIR)/RenderTexture.cpp \
	$(SRCDIR)/RSP_Parser.cpp \
	$(SRCDIR)/RSP_S2DEX.cpp \
	$(SRCDIR)/Texture.cpp \
	$(SRCDIR)/TextureFilters.cpp \
	$(SRCDIR)/TextureFilters_2xsai.cpp \
	$(SRCDIR)/TextureFilters_hq2x.cpp \
	$(SRCDIR)/TextureFilters_hq4x.cpp \
	$(SRCDIR)/TextureManager.cpp \
	$(SRCDIR)/VectorMath.cpp \
	$(SRCDIR)/Video.cpp \
	$(SRCDIR)/osal_dynamiclib_unix.c \
	$(SRCDIR)/osal_files_unix.c

# generate a list of object files build, make a temporary directory for them
OBJECTS := $(patsubst $(SRCDIR)/%.c, $(OBJDIR)/%.o, $(filter %.c, $(SOURCE)))
OBJECTS += $(patsubst $(SRCDIR)/%.cpp, $(OBJDIR)/%.o, $(filter %.cpp, $(SOURCE)))
OBJDIRS = $(dir $(OBJECTS))
$(shell $(MKDIR) $(OBJDIRS))

# build targets
TARGET = mupen64plus-ae-video-rice$(POSTFIX).$(SO_EXTENSION)

targets:
	@echo "Mupen64plus-video-rice N64 Graphics plugin makefile (Linux ARMv7 + GLES2 + MALI). "
	@echo "  Targets:"
	@echo "    all           == Build Mupen64plus-video-rice plugin"
	@echo "    clean         == remove object files"
	@echo "    rebuild       == clean and re-build all"
	@echo "    install       == Install Mupen64Plus-video-rice plugin"
	@echo "    uninstall     == Uninstall Mupen64Plus-video-rice plugin"
	@echo "  Options:"
	@echo "    APIDIR=path   == path to find Mupen64Plus Core headers"
	@echo "    OPTFLAGS=flag == compiler optimization (default: -O3 -flto)"
	@echo "    WARNFLAGS=flag == compiler warning levels (default: -Wall)"
	@echo "    POSTFIX=name  == String added to the name of the the build (default: '')"
	@echo "  Install Options:"
	@echo "    PREFIX=path   == install/uninstall prefix (default: /usr/local)"
	@echo "    SHAREDIR=path == path to install shared data files (default: PREFIX/share/mupen64plus)"
	@echo "    LIBDIR=path   == library prefix (default: PREFIX/lib)"
	@echo "    PLUGINDIR=path == path to install plugin libraries (default: LIBDIR/mupen64plus)"
	@echo "    DESTDIR=path  == path to prepend to all installation paths (only for packagers)"
	@echo "  Debugging Options:"
	@echo "    DEBUG=1       == add debugging symbols"
	@echo "    V=1           == show verbose compiler output"

all: $(TARGET)

install: $(TARGET)
	$(INSTALL) -d "$(DESTDIR)$(PLUGINDIR)"
	$(INSTALL) -m 0644 $(INSTALL_STRIP_FLAG) $(TARGET) "$(DESTDIR)$(PLUGINDIR)"
	$(INSTALL) -d "$(DESTDIR)$(SHAREDIR)"
	$(INSTALL) -m 0644 "../../data/RiceVideoLinux.ini" "$(DESTDIR)$(SHAREDIR)"

uninstall:
	$(RM) "$(DESTDIR)$(PLUGINDIR)/$(TARGET)"
	$(RM) "$(DESTDIR)$(SHAREDIR)/RiceVideoLinux.ini"

clean:
	$(RM) -r $(OBJDIR) $(TARGET)

rebuild: clean all

# build dependency files
CFLAGS += -MD -MP
-include $(OBJECTS:.o=.d)

CXXFLAGS += $(CFLAGS)

# standard build rules
$(OBJDIR)/%.o: $(SRCDIR)/%.c
	$(COMPILE.c) -o $@ $<

$(OBJDIR)/%.o: $(SRCDIR)/%.cpp
	$(COMPILE.cc) -o $@ $<

$(TARGET): $(OBJECTS)
	$(LINK.o) $^ $(LOADLIBES) $(LDLIBS) -o $@

.PHONY: all clean install uninstall targets