KERNEL_DIR=/usr/src/linux-$(shell uname -r)

TARGET=ipt_account.o

obj-m:=$(TARGET)
ipt_account-objs:=$(OBJS)
TARGETS := $(obj-m:.o=.ko)

DESTDIR_ROOT:=/lib/modules/$(shell uname -r)
DESTDIR:=$(DESTDIR_ROOT)/kernel/net/ipv4/netfilter

all:		
	$(MAKE) -C $(KERNEL_DIR) SUBDIRS=$$PWD modules

$(TARGETS): all

install: 	$(TARGETS)
		@test -d $(DESTDIR) || mkdir $(DESTDIR)
		@set -x;install -d $(DESTDIR) && install -p -m 644 $(TARGETS) $(DESTDIR)
uninstall: 	
		cd $(DESTDIR);rm -fv $(TARGETS)

clean:		
		rm -f .*.cmd *.mod.c *.ko *.o *~ core $(TARGETS)
		rm -fr .tmp_versions
