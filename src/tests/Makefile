# -*- text -*-
##
## Makefile -- Build and run tests for the server.
##
##	http://www.freeradius.org/
##	$Id$
##
#
include ../../Make.inc

ifeq "$(BOILER)" "yes"
	BUILD_PATH := ../../build
	BIN_PATH := $(BUILD_PATH)/bin/local
	LIB_PATH := $(BUILD_PATH)/lib/local/.libs/
	DYLD_LIBRARY_PATH := $(DYLD_LIBRARY_PATH):$(LIB_PATH)
else
	BIN_PATH := ../main
	DYLD_LIBRARY_PATH := $(DYLD_LIBRARY_PATH):../lib/.libs/:../modules/lib/.libs/
endif

export DYLD_LIBRARY_PATH

RADDB_PATH := ../../raddb

TESTS	= user_password chap mschapv1 digest-01/digest* test.example.com \
	wimax

PORT	 = 12340
ACCTPORT = $(shell expr $(PORT) + 1)

#	example.com stripped.example.com

EAPOL_TEST = eapol_test

EAP_TLS_TESTS = eap-ttls-pap.conf eap-mschapv2.conf \
	      eap-ttls-mschapv2.conf peap-mschapv2.conf

SECRET	= testing123

.PHONY: all eap dictionary clean

#
#	Build the directory for testing the server
#
all: tests

clean:
	@rm -f $(RADDB_PATH)/test.conf test.conf dictionary

dictionary:
	@echo "# test dictionary not install.  Delete at any time." > dictionary
	@echo '$$INCLUDE ' $(top_builddir)/share/dictionary >> dictionary
	@echo '$$INCLUDE ' $(top_builddir)/src/tests/dictionary.test >> dictionary

test.conf: dictionary
	@echo "# test configuration file.  Do not install.  Delete at any time." > test.conf
	@echo "libdir =" $(top_builddir)/src/modules/lib >> test.conf
	@echo "testdir =" $(top_builddir)/src/tests/ >> test.conf
	@echo 'dictionary = $${testdir}' >> test.conf
	@echo 'logdir = $${testdir}' >> test.conf
	@echo 'radacctdir = $${testdir}' >> test.conf
	@echo 'pidfile = $${testdir}/radiusd.pid' >> test.conf
	@echo '$$INCLUDE radiusd.conf' >> test.conf
	@echo '$$INCLUDE $${testdir}/config/' >> test.conf

radiusd.pid: $(RADDB_PATH)/test.conf test.conf raddb
	@$(BIN_PATH)/radiusd -txxl `pwd`/radius.log -md $(RADDB_PATH)/ -n test -i 127.0.0.1 -p $(PORT)

# We can't make this depend on radiusd.pid, because then make will create
# radiusd.pid when we make radiusd.kill, which we don't want.
.PHONY: radiusd.kill
radiusd.kill:
	@if [ -f radiusd.pid ]; then \
		(kill -TERM `cat radiusd.pid` >/dev/null 2>&1) || exit 0; \
	fi
	@rm -f radiusd.pid

# As test.conf includes radiusd.conf and as radiusd.conf includes everything in
# mods-enabled and sites-enable we need to ensure those directories have been
# created and are up to date.
.PHONY: raddb
raddb:
	@$(MAKE) -C $(RADDB_PATH) mods-enabled sites-enabled

#  Link from the main database directory to here
$(RADDB_PATH)/test.conf: test.conf
	@[ -f $(RADDB_PATH)/test.conf ] || ln -s ../src/tests/test.conf $(RADDB_PATH)/

# kill the server (if it's running)
# start the server
# run the tests (ignoring any failures)
# kill the server
# remove the changes to raddb/
tests: $(RADDB_PATH)/test.conf radiusd.kill
	@chmod a+x runtests.sh
	@rm -f radius.log
	@$(MAKE) radiusd.pid
	@BIN_PATH="$(BIN_PATH)" ./runtests.sh $(TESTS)
	@$(MAKE) radiusd.kill
	@rm -f $(RADDB_PATH)/test.conf

tests.eap: $(RADDB_PATH)/test.conf radiusd.kill
	@chmod a+x runtests.sh
	@rm -f radius.log
	@$(MAKE) radiusd.pid
	@$(MAKE) eap
	@$(MAKE) radiusd.kill
	@rm -f $(RADDB_PATH)/test.conf

eap: $(EAP_TLS_TESTS)
	for x in $(EAP_TLS_TESTS); do \
		$(EAPOL_TEST) -c $$x -p $(PORT) -s $(SECRET); \
	done

md5:
	$(EAPOL_TEST) -c eap-md5.conf -s $(SECRET) 

tls:
	$(EAPOL_TEST) -c eap-ttls-tls.conf -s $(SECRET)

ttls:
	$(EAPOL_TEST) -c eap-ttls-pap.conf -s $(SECRET)

peap:
	$(EAPOL_TEST) -c peap-mschapv2.conf -s $(SECRET)

leap:
	$(EAPOL_TEST) -c leap.conf -s $(SECRET)

ATTRS	:= rfc.txt errors.txt extended.txt lucent.txt wimax.txt

attrs:	$(ATTRS) ../main/radattr
	../main/radattr -d ../../share rfc.txt

${LIBRADIUS}: $(wildcard ../include/*.h) $(wildcard ../lib/*.c)
	$(MAKE) -C ../lib all

../main/radattr: ${LIBRADIUS} ../main/radattr.c
	$(MAKE) -C ../main radattr
