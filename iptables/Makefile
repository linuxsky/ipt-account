KERNEL_DIR=/usr/src/linux-$(shell uname -r)
IPTABLES_LIB_DIR=/lib/iptables/
IPTABLES_VERSION=$(shell iptables --version | sed 's/^iptables v//')

CC=cc
LD=cc
# NOTE: if you are using ubuntu, you probably need to add -fno-stack-protector
# NOTE: if you are using iptables 1.4.0 (only this version) you probably need to add -DXTABLES_VERSION=\"1.4.0\"
CFLAGS=-O2 -Wall -Wunused -I$(KERNEL_DIR) -Iinclude/ -DIPTABLES_VERSION=\"$(IPTABLES_VERSION)\" -g -DDEBUG -fPIC
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

