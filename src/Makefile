COMMIT_REV ?= $(shell dpkg-query -W -f='$${Version}' flashcache-dkms | awk -F "-" '{print $$1}' | cut -d\: -f2)
KERNEL_SOURCE_VERSION ?= $(shell uname -r)
KERNEL_TREE ?= /lib/modules/$(KERNEL_SOURCE_VERSION)/build

EXTRA_CFLAGS += -I$(KERNEL_TREE)/drivers/md -I./ -DCOMMIT_REV="\"$(COMMIT_REV)\""
EXTRA_CFLAGS += -I$(KERNEL_TREE)/include/ -I$(KERNEL_TREE)/include/linux 

# Check for RHEL/CentOS
RHEL5_VER ?= $(shell  if [ -e /etc/redhat-release ]; then grep " 5\\.[0-9]" /etc/redhat-release; else false; fi)
RHEL5_SETUP :=
ifneq "$(RHEL5_VER)" ""
	RHEL5_SETUP := rhel5-setup
	RHEL5_SPEC := /usr/src/redhat/SPECS/kernel.spec
	RHEL5_TREE := /usr/src/redhat/BUILD/kernel-2.6.18/linux-$(shell uname -r).$(shell uname -i)
	RHEL5_SRC := /usr/src/kernels/$(shell uname -r)-$(shell uname -i)
	KERNEL_TREE := $(RHEL5_TREE)
endif

# Check for RHEL/CentOS 7
RHEL7_VER ?= $(shell if [ -e /etc/redhat-release ]; then grep " 7\\.[0-9]" /etc/redhat-release; else false; fi)
ifneq "$(RHEL7_VER)" ""
        RHEL7_TREE := /usr/src/kernels/$(shell uname -r)
        KERNEL_TREE := $(RHEL7_TREE)
endif

# Check for OpenVZ (/proc/vz)
OPENVZ_VER ?= $(shell if [ -e /proc/vz ]; then grep 5.[0-9] /etc/redhat-release; else false; fi)
ifneq "$(OPENVZ_VER)" ""
        RHEL5_SPEC := /usr/src/redhat/SPECS/kernel-ovz.spec
        RHEL5_TREE := /usr/src/redhat/BUILD/ovzkernel-2.6.18/linux-$(shell uname -r).$(shell uname -i)
        KERNEL_TREE := $(RHEL5_TREE)
endif

obj-m += flashcache.o
flashcache-objs := flashcache_conf.o flashcache_main.o flashcache_subr.o flashcache_ioctl.o flashcache_procfs.o flashcache_reclaim.o flashcache_kcopy.o

.PHONY: all
all: modules

.PHONY:    modules
modules: $(RHEL5_SETUP)
	make -C $(KERNEL_TREE) M=$(PWD) modules V=0

.PHONY:    utils
utils:
	make -C utils all

.PHONY: modules_install
modules_install: modules
	install -o root -g root -m 0755 -d $(DESTDIR)/lib/modules/$(KERNEL_SOURCE_VERSION)/extra/flashcache/
	install -o root -g root -m 0755 flashcache.ko $(DESTDIR)/lib/modules/$(KERNEL_SOURCE_VERSION)/extra/flashcache/
	depmod -a $(KERNEL_SOURCE_VERSION)

.PHONY: utils_install
utils_install: utils
	make -C utils install

.PHONY: ocf_install
ocf_install:
	make -C ocf install

.PHONY: install
install: modules_install ocf_install

.PHONY: clean
clean:
	make -C $(KERNEL_TREE) M=$(PWD) clean
	make -C utils clean

.PHONY: rhel5-setup
rhel5-setup: $(RHEL5_TREE)
	make -C $(RHEL5_TREE) oldconfig ; \
	make -C $(RHEL5_TREE) prepare modules_prepare
	ln -s -f $(RHEL5_SRC)/Module.symvers $(RHEL5_TREE)/Module.symvers

$(RHEL5_TREE):
	rpmbuild -bp --target=`uname -m` $(RHEL5_SPEC) 2>&1 | tee `dirname $(RHEL5_SPEC)`/prep.log
