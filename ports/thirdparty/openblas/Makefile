include ../../../mk/pitchfork.mk

# FC check
include ../../../mk/chkfc.mk

# Local variables
_NAME   = OpenBLAS-0.2.20
_URL    = https://github.com/xianyi/OpenBLAS/archive
_WRKSRC = $(WORKDIR)/$(_NAME)
_FILE   = v0.2.20.tar.gz

# Local works
ifeq ($(origin HAVE_OPENBLAS),undefined)
do-extract: $(_WRKSRC)
$(_WRKSRC): | do-fetch
	tar zxf $(_FILE) -C $(WORKDIR)
do-fetch: $(_FILE)
$(_FILE):
	$(PFHOME)/bin/pitchfork fetch --url $(_URL)/$@
do-build: $(_WRKSRC)/libopenblas.$(DYLIB)
$(_WRKSRC)/libopenblas.$(DYLIB): | do-extract
	$(MAKE) -C $(_WRKSRC) ONLY_CBLAS=1 MAKE_NB_JOBS=2 2>&1 | tee build.log | $(PFHOME)/bin/activity || $(PFHOME)/bin/diewith build.log
do-install: $(PREFIX)/var/pkg/$(_NAME)
$(PREFIX)/var/pkg/$(_NAME): | do-build
	rm -rf $(STAGING)/$(_NAME)
	$(MAKE) -j1 -C $(_WRKSRC) PREFIX=$(STAGING)/$(_NAME) install >& install.log || $(PFHOME)/bin/diewith install.log
	cd $(STAGING)/$(_NAME) && tar cf - * | tar xf - -C $(PREFIX)
	find $(STAGING)/$(_NAME) ! -type d|awk -F '$(STAGING)/$(_NAME)/' '{print $$2}' > $@
else
ifeq ($(wildcard $(HAVE_OPENBLAS)/libopenblas*),)
ifeq ($(wildcard $(HAVE_OPENBLAS)/lib/libopenblas*),)
$(error library under $(HAVE_OPENBLAS) not found)
else
_OPENBLAS = $(HAVE_OPENBLAS)/lib
endif
else
_OPENBLAS = $(HAVE_OPENBLAS)
endif
_NAME     = OpenBLAS
do-install:
	rm -rf $(STAGING)/$(_NAME)
	mkdir -p $(STAGING)/$(_NAME)/lib
	rsync -avx $(_OPENBLAS)/libopenblas*          $(STAGING)/$(_NAME)/lib/
	cd $(STAGING)/$(_NAME) && tar cf - * | tar xf - -C $(PREFIX)
	find $(STAGING)/$(_NAME) ! -type d|awk -F '$(STAGING)/$(_NAME)/' '{print $$2}' > $(PREFIX)/var/pkg/$(_NAME)
endif
do-clean: do-flush
do-distclean: do-flush
do-flush:
	rm -rf $(_WRKSRC)
	rm -f build.log install.log
