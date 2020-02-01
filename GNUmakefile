# SPDX-License-Identifier: MIT
# Copyright (C) 2018 Jason A. Donenfeld <Jason@zx2c4.com>. All Rights Reserved.

# Device configuration
IMAGE			?= Image.gz-dtb
DEFCONFIG		?= jasmine_defconfig

# Toolchain configuration
TOOLCHAIN_HOST		:= https://github.com/alesaiko
GCC_VERSION		?= 1182d9bbb690

# Do not print directory changes
MAKEFLAGS += --no-print-directory

# Use all CPU threads
ifeq ($(findstring -j,$(MAKEFLAGS)),)
MAKEFLAGS += -j$(shell nproc)
endif

default: all

define toolchain
toolchains/$(1)/$(2)/.prepared:
	@printf "\e[1;32m  - Downloading $(1) [$(2)]...\e[0m\n"
	@mkdir -p toolchains
	@git clone $(TOOLCHAIN_HOST)/$(1) -b $(2) toolchains/$(1)/$(2)
	@touch $$@
all: toolchains/$(1)/$(2)/.prepared
%:: toolchains/$(1)/$(2)/.prepared
endef

# Arguments to be passed to super Makefile
MAKE_ARGS :=

ifeq ($(CROSS_COMPILE),)
$(eval $(call toolchain,aarch64-linux-android-4.9,$(GCC_VERSION)))

# Use Android GCC 4.9 by default
export CROSS_COMPILE=$(CURDIR)/toolchains/aarch64-linux-android-4.9/$(GCC_VERSION)/bin/aarch64-linux-android-
endif

export ARCH=arm64
export SUBARCH=arm64

SUBMODULES := $(shell [ -e .git ] && git config -f .gitmodules --get-regexp '\.path' | sed 's/^[^ ]\+ \(.*\)/\1\/.git/')

all: out/.config $(SUBMODULES)
	@printf "\e[1;32m  - Building kernel...\e[0m\n"
	@$(MAKE) -f Makefile O=out $(MAKE_ARGS)

out/.config: $(SUBMODULES)
	@printf "\e[1;32m  - Generating configuration...\e[0m\n"
	@$(MAKE) -f Makefile O=out $(MAKE_ARGS) $(DEFCONFIG)

clean-toolchains:
	@printf "\e[1;32m  - Cleaning toolchains...\e[0m\n"
	@$(RM) -rf toolchains

fastboot: all
	@printf "\e[1;32m  - Booting out/arch/$(ARCH)/boot/$(IMAGE)...\e[0m\n"
	@fastboot boot out/arch/$(ARCH)/boot/$(IMAGE)
	@printf "\e[36m  - Waiting for device to boot...\e[0m\n"
	@adb wait-for-device
	@printf "\e[1;32m  - Requesting dmesg...\e[0m\n"
	@adb shell su -c dmesg

%:: out/.config $(SUBMODULES)
	@printf "\e[1;32m  - Kernel $@ target...\e[0m\n"
	@$(MAKE) -f Makefile O=out $(MAKE_ARGS) $@

ifneq ($(SUBMODULES),)
$(SUBMODULES):
	@printf "\e[1;32m  - Initializing and updating submodules...\e[0m\n"
	@git submodule init
	@git submodule update
endif

.PHONY: clean-toolchains all default fastboot GNUmakefile
