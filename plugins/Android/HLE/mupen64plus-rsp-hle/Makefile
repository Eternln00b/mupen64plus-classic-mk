# ----------------------------------------------------------------------
# N00b-Power SDK configuration
# ----------------------------------------------------------------------
SDK_ROOT    := /home/maximilien/projects/N00b-Power.SDK
SDK_SYSROOT := $(SDK_ROOT)/sysroot
LINARO_CC   := $(SDK_ROOT)/gcc-linaro/bin

CROSS_COMPILE ?= $(LINARO_CC)/arm-linux-gnueabihf-
CC            := $(CROSS_COMPILE)gcc
ARCH_FLAGS    := -marm -mtune=cortex-a7 -mfpu=neon-vfpv4 -mfloat-abi=hard

# ----------------------------------------------------------------------
# Mupen64Plus RSP HLE Makefile (Linux ARMv7 uniquement)
# ----------------------------------------------------------------------

# OS : Linux uniquement
OS := LINUX
SO_EXTENSION := so
SHARED := -shared

# Architecture : ARM 32-bit uniquement
HOST_CPU := armv7
NO_ASM := 1
CPU := ARM
ARCH_DETECTED := 32BITS
PIC := 1

SRCDIR = ../../src
OBJDIR = _obj$(POSTFIX)

# base CFLAGS, LDLIBS, and LDFLAGS
OPTFLAGS ?= -flto -O2 -fno-stack-protector -Wno-unused-function -Wno-deprecated-declarations -Wno-overlength-strings
WARNFLAGS ?= -Wall
CFLAGS += $(OPTFLAGS) $(WARNFLAGS) $(ARCH_FLAGS) -ffast-math -fno-strict-aliasing -fvisibility=hidden -I$(SRCDIR)
LDFLAGS += $(SHARED)

# Position independent code
CFLAGS += -fPIC

# set special flags per-system (Linux uniquement)
LDFLAGS += -Wl,-version-script,$(SRCDIR)/rsp_api_export.ver
LDLIBS += -ldl

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

# reduced compile output
ifneq ($(findstring $(MAKEFLAGS),s),s)
ifndef V
	Q_CC  = @echo '    CC  '$@;
	Q_LD  = @echo '    LD  '$@;
endif
endif

RM       ?= rm -f
INSTALL  ?= install
MKDIR ?= mkdir -p
COMPILE.c = $(Q_CC)$(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
LINK.o = $(Q_LD)$(CC) $(CFLAGS) $(LDFLAGS) $(TARGET_ARCH)

# set special flags for given Makefile parameters
ifeq ($(DEBUG),1)
  CFLAGS += -g
  INSTALL_STRIP_FLAG ?= 
else
  CFLAGS += -DNDEBUG
  INSTALL_STRIP_FLAG ?= --strip-program=$(CROSS_COMPILE)strip -s
endif

# set installation options
PREFIX ?= ${SDK_SYSROOT}/usr
LIBDIR ?= $(PREFIX)/lib
PLUGINDIR ?= $(LIBDIR)/mupen64plus

# enable/disable task dumping support
ifeq ($(DUMP), 1)
	CFLAGS += -DENABLE_TASK_DUMP
endif

# list of source files to compile
SOURCE = \
	$(SRCDIR)/alist.c \
	$(SRCDIR)/alist_audio.c \
	$(SRCDIR)/alist_naudio.c \
	$(SRCDIR)/alist_nead.c \
	$(SRCDIR)/audio.c \
	$(SRCDIR)/cicx105.c \
	$(SRCDIR)/hle.c \
	$(SRCDIR)/hvqm.c \
	$(SRCDIR)/jpeg.c \
	$(SRCDIR)/memory.c \
	$(SRCDIR)/mp3.c \
	$(SRCDIR)/musyx.c \
	$(SRCDIR)/re2.c \
	$(SRCDIR)/plugin.c \
	$(SRCDIR)/osal_dynamiclib_unix.c

# generate a list of object files build
OBJECTS := $(patsubst $(SRCDIR)/%.c, $(OBJDIR)/%.o, $(filter %.c, $(SOURCE)))
OBJDIRS = $(dir $(OBJECTS))
$(shell $(MKDIR) $(OBJDIRS))

# build targets
TARGET = mupen64plus-ae-rsp-hle$(POSTFIX).$(SO_EXTENSION)

targets:
	@echo "Mupen64Plus-rsp-hle makefile (Linux ARMv7). "
	@echo "  Targets:"
	@echo "    all           == Build Mupen64Plus rsp-hle plugin"
	@echo "    clean         == remove object files"
	@echo "    rebuild       == clean and re-build all"
	@echo "    install       == Install Mupen64Plus rsp-hle plugin"
	@echo "    uninstall     == Uninstall Mupen64Plus rsp-hle plugin"
	@echo "  Options:"
	@echo "    APIDIR=path   == path to find Mupen64Plus Core headers"
	@echo "    OPTFLAGS=flag == compiler optimization (default: -O3 -flto)"
	@echo "    WARNFLAGS=flag == compiler warning levels (default: -Wall)"
	@echo "    POSTFIX=name  == String added to the name of the the build (default: '')"
	@echo "    DUMP=(1|0)    == Enable/Disable unknown task dumping (default: 0)"
	@echo "  Install Options:"
	@echo "    PREFIX=path   == install/uninstall prefix (default: /usr/local)"
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

uninstall:
	$(RM) "$(DESTDIR)$(PLUGINDIR)/$(TARGET)"

clean:
	$(RM) -r $(OBJDIR) $(TARGET)

rebuild: clean all

# build dependency files
CFLAGS += -MD -MP
-include $(OBJECTS:.o=.d)

# standard build rules
$(OBJDIR)/%.o: $(SRCDIR)/%.c
	$(COMPILE.c) -o $@ $<

$(TARGET): $(OBJECTS)
	$(LINK.o) $^ $(LOADLIBES) $(LDLIBS) -o $@

.PHONY: all clean install uninstall targets