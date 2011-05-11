KERNEL_DIR=/usr/src/linux-$(shell uname -r)
IPTABLES_VERSION=$(shell iptables --version | sed 's/^iptables v//')

ifneq (,$(findstring 1.4,$(IPTABLES_VERSION)))
IPTABLES_LIB_DIR=/lib/xtables/
else
IPTABLES_LIB_DIR=/lib/xtables/
endif

CC=cc
LD=cc
# NOTE: if you are using ubuntu, you probably need to add -fno-stack-protector
ifneq (,$(findstring 1.4,$(IPTABLES_VERSION)))
CFLAGS=-O2 -Wall -Wunused -I$(KERNEL_DIR) -Iinclude/ -DXTABLES_VERSION=\"$(IPTABLES_VERSION)\" -g -DDEBUG -fPIC
else
CFLAGS=-O2 -Wall -Wunused -I$(KERNEL_DIR) -Iinclude/ -DIPTABLES_VERSION=\"$(IPTABLES_VERSION)\" -g -DDEBUG -fPIC
endif
LDFLAGS=-shared

OBJS=libipt_account_sh.o
LIBS=libipt_account.so

all : $(OBJS) $(LIBS)

$(OBJS): libipt_account.c
	$(CC) $(CFLAGS) -o $@ -c $^

$(LIBS): libipt_account_sh.o 
	$(CC) $(LDFLAGS) -o $@ $^

install:
	cp libipt_account.so $(IPTABLES_LIB_DIR)

uninstall:
	rm -fv $(IPTABLES_LIB_DIR)$(LIBS)

clean:
	rm -fv $(OBJS) $(LIBS) *.o *.so *.so.*

