

ifeq ($(NEOTONIC_ROOT),)
NEOTONIC_ROOT = ..
endif

include $(NEOTONIC_ROOT)/rules.mk

CGI_LIB = $(LIB_DIR)libneo_cgi.a
CGI_SRC = cgiwrap.c cgi.c html.c date.c rfc2388.c
CGI_OBJ = $(CGI_SRC:%.c=%.o)

STATIC_EXE = cs_static.cgi
STATIC_SRC = static.c
STATIC_OBJ = $(STATIC_SRC:%.c=%.o)
STATIC_CSO = $(STATIC_EXE:%.cgi=%.cso)

CGICSTEST_EXE = cgi_cstest
CGICSTEST_SRC = cgi_cstest.c
CGICSTEST_OBJ = $(CGICSTEST_SRC:%.c=%.o)

CGITEST_EXE = cgi_test
CGITEST_SRC = cgi_test.c
CGITEST_OBJ = $(CGITEST_SRC:%.c=%.o)

DLIBS += -lneo_cgi -lneo_cs -lneo_utl -lstreamhtmlparser # -lefence

LDRUN = LD_LIBRARY_PATH=$(LD_LIBRARY_PATH):../libs

TARGETS = $(CGI_LIB) $(STATIC_EXE) $(STATIC_CSO) $(CGICSTEST_EXE) \
	$(CGITEST_EXE) test

CGI_CS_TESTS = test_cgi_funcs.cs test_cgi_escaping.cs
CGI_CS_AUTO_TESTS = test_cgi_auto.cs

all: $(TARGETS)

$(CGI_LIB): $(CGI_OBJ)
	$(AR) $@ $(CGI_OBJ)
	$(RANLIB) $@

$(STATIC_EXE): $(STATIC_OBJ) $(DEP_LIBS)
	$(LD) $@ $(STATIC_OBJ) $(LDFLAGS) $(DLIBS) $(LIBS)

$(STATIC_CSO): $(STATIC_OBJ) $(DEP_LIBS)
	$(LDSHARED) -o $@ $(STATIC_OBJ) $(LDFLAGS) $(DLIBS) $(LIBS)

$(CGICSTEST_EXE): $(CGICSTEST_OBJ) $(DEP_LIBS)
	$(LD) $@ $(CGICSTEST_OBJ) $(LDFLAGS) $(DLIBS) $(LIBS)

$(CGITEST_EXE): $(CGITEST_OBJ) $(DEP_LIBS)
	$(LD) $@ $(CGITEST_OBJ) $(LDFLAGS) $(DLIBS) $(LIBS)

## BE VERY CAREFUL WHEN REGENERATING THESE
gold: $(CGICSTEST_EXE)
	@for test in $(CGI_CS_TESTS); do \
		rm -f $$test.gold; \
		$(LDRUN) ./cgi_cstest ../cs/test.hdf $$test > $$test.gold; \
	done
	@for test in $(CGI_CS_AUTO_TESTS); do \
		rm -f $$test.gold; \
		$(LDRUN) ./cgi_cstest test_cgi_auto.hdf $$test > $$test.gold; \
	done
	@echo "Generated Gold Files"

test: $(CGICSTEST_EXE) $(CGI_CS_TESTS) $(CGITEST_EXE) $(CGI_CS_AUTO_TESTS)
	@echo "Running cgi cs regression tests"
	@failed=0; \
	for test in $(CGI_CS_TESTS); do \
		rm -f $$test.out; \
		$(LDRUN) ./cgi_cstest ../cs/test.hdf $$test > $$test.out 2>&1; \
		diff $$test.out $$test.gold 2>&1 > /dev/null; \
		return_code=$$?; \
		if [ $$return_code -ne 0 ]; then \
		  diff $$test.gold $$test.out > $$test.err; \
		  echo "Failed Regression Test: $$test"; \
		  echo "  See $$test.out and $$test.err"; \
		  failed=1; \
		fi; \
	done; \
	for test in $(CGI_CS_AUTO_TESTS); do \
		rm -f $$test.out; \
		$(LDRUN) ./cgi_cstest test_cgi_auto.hdf $$test > $$test.out 2>&1; \
		diff $$test.out $$test.gold 2>&1 > /dev/null; \
		return_code=$$?; \
		if [ $$return_code -ne 0 ]; then \
		  diff $$test.gold $$test.out > $$test.err; \
		  echo "Failed Regression Test: $$test"; \
		  echo "  See $$test.out and $$test.err"; \
		  failed=1; \
		fi; \
	done; \
	$(LDRUN) ./cgi_test; \
	return_code=$$?; \
	if [ $$return_code -ne 0 ]; then \
	  echo "Failed standalone cgi_test"; \
	  failed=1; \
	fi; \
	if [ $$failed -eq 1 ]; then \
	  exit 1; \
	fi;
	@touch test
	@echo "Passed"

install: all
	$(NEOTONIC_ROOT)/mkinstalldirs $(DESTDIR)$(cs_includedir)/cgi
	$(INSTALL) -m 644 cgi.h $(DESTDIR)$(cs_includedir)/cgi
	$(INSTALL) -m 644 cgiwrap.h $(DESTDIR)$(cs_includedir)/cgi
	$(INSTALL) -m 644 date.h $(DESTDIR)$(cs_includedir)/cgi
	$(INSTALL) -m 644 html.h $(DESTDIR)$(cs_includedir)/cgi
	$(INSTALL) -m 644 $(CGI_LIB) $(DESTDIR)$(libdir)
	$(INSTALL) $(STATIC_EXE) $(DESTDIR)$(bindir)

clean:
	$(RM) *.o

distclean:
	$(RM) Makefile.depends $(TARGETS) *.o
