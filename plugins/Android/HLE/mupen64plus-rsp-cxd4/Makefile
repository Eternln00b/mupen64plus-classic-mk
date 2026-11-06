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
# Mupen64Plus RSP CXD4 Makefile (Linux ARMv7 uniquement)
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

# Optimisations NEON pour ARMv7
NEON = 1

# base CFLAGS, LDLIBS, and LDFLAGS
OPTFLAGS ?= -flto -O2 -fno-stack-protector -Wno-unused-function -Wno-deprecated-declarations -Wno-overlength-strings
WARNFLAGS ?= -Wall
CFLAGS += $(OPTFLAGS) $(WARNFLAGS) $(ARCH_FLAGS) -fvisibility=hidden -I../../
CPPFLAGS += -DM64P_PLUGIN_API
LDFLAGS += $(SHARED)

# NEON (traduction SSE -> NEON)
ifeq ($(NEON), 1)
    CFLAGS   += -mfpu=neon-vfpv4
    CPPFLAGS += -DUSE_SSE2NEON
endif

# Position independent code (obligatoire pour une .so)
CFLAGS += -fPIC
LDFLAGS += -fPIC

# set special flags per-system (Linux uniquement)
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

# reduced compile output when running make without V=1
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

SRCDIR = ../../
OBJDIR = _obj$(POSTFIX)

# list of source files to compile
SOURCE = \
	$(SRCDIR)/su.c \
	$(SRCDIR)/vu/add.c \
	$(SRCDIR)/vu/divide.c \
	$(SRCDIR)/vu/logical.c \
	$(SRCDIR)/vu/multiply.c \
	$(SRCDIR)/vu/select.c \
	$(SRCDIR)/vu/vu.c \
	$(SRCDIR)/module.c \
	$(SRCDIR)/osal_dynamiclib_unix.c

# generate a list of object files build, make a temporary directory for them
OBJECTS := $(patsubst $(SRCDIR)/%.c, $(OBJDIR)/%.o, $(filter %.c, $(SOURCE)))
OBJDIRS = $(dir $(OBJECTS))
$(shell $(MKDIR) $(OBJDIRS))

# build targets
TARGET = mupen64plus-ae-rsp-cxd4$(POSTFIX).$(SO_EXTENSION)

targets:
	@echo "Mupen64Plus-rsp-cxd4 makefile (Linux ARMv7). "
	@echo "  Targets:"
	@echo "    all           == Build Mupen64Plus rsp-cxd4 plugin"
	@echo "    clean         == remove object files"
	@echo "    rebuild       == clean and re-build all"
	@echo "    install       == Install Mupen64Plus rsp-cxd4 plugin"
	@echo "    uninstall     == Uninstall Mupen64Plus rsp-cxd4 plugin"
	@echo "  Options:"
	@echo "    APIDIR=path   == path to find Mupen64Plus Core headers"
	@echo "    OPTFLAGS=flag == compiler optimization (default: -O3 -flto)"
	@echo "    WARNFLAGS=flag == compiler warning levels (default: -Wall)"
	@echo "    HLEVIDEO=(1|0) == Move task of gfx emulation to a HLE video plugins"
	@echo "    POSTFIX=name  == String added to the name of the the build (default: '')"
	@echo "    NEON=(1|0)    == Optimize for NEON technology version (default: 1)"
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