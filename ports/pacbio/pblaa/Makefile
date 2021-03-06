include ../../../mk/pitchfork.mk

# Local variables
_NAME             = pblaa
_WRKSRC           = $(WORKDIR)/$(_NAME)
$(_NAME)_VERSION ?= HEAD
_REVISION         = $(shell cd $(_WRKSRC) && $(GIT) rev-parse --short $($(_NAME)_VERSION) || true)

ifeq ($(origin $(_NAME)_REPO),undefined)
    $(error "You need to set $(_NAME)_REPO")
endif

# Local works
do-fetch: $(_WRKSRC)
$(_WRKSRC):
ifeq ($(wildcard $($(_NAME)_REPO)),)
	$(GIT) clone $($(_NAME)_REPO) $@
	cd $(_WRKSRC) && $(GIT) checkout $($(_NAME)_VERSION)
else
	ln -sfn $($(_NAME)_REPO) $(_WRKSRC)
endif
ifneq ($(wildcard $(PREFIX)/lib/libpbbam.$(DYLIB)),)
PacBioBAM_LIBRARIES = $(PREFIX)/lib/libpbbam.$(DYLIB)
else
PacBioBAM_LIBRARIES = $(PREFIX)/lib/libpbbam.a
endif
do-config: $(_WRKSRC)/_build/Makefile
$(_WRKSRC)/_build/Makefile: | do-fetch
	mkdir -p $(_WRKSRC)/_build
	cd $(_WRKSRC)/_build && \
        $(CMAKE) \
            -DZLIB_INCLUDE_DIRS=$(ZLIB_ROOT)/include \
            -DZLIB_LIBRARIES=$(ZLIB_ROOT)/lib/libz.$(DYLIB) \
            -DHTSLIB_INCLUDE_DIRS=$(PREFIX)/src/htslib/include \
            -DHTSLIB_LIBRARIES=$(PREFIX)/src/htslib/lib/libhts.a \
            -DPacBioBAM_INCLUDE_DIRS=$(PREFIX)/include \
            -DPacBioBAM_LIBRARIES=$(PacBioBAM_LIBRARIES) \
            -DSEQAN_INCLUDE_DIRS=$(PREFIX)/include \
            -DSEQAN_RootDir=$(PREFIX) \
            -DBoost_INCLUDE_DIRS=$(BOOST_ROOT)/include \
            -DPacBioLAA_UNY=$(_WRKSRC)/../unanimity \
            -DCMAKE_BUILD_TYPE=RelWithDebInfo \
            -DCMAKE_SKIP_BUILD_RPATH=TRUE \
            -Dpbcopper_INCLUDE_DIRS=$(PREFIX)/include \
            -Dpbcopper_LIBRARIES=$(PREFIX)/lib/libpbcopper.a \
            ..
do-build: $(_WRKSRC)/bin/laa
$(_WRKSRC)/bin/laa: | do-config
	cd $(_WRKSRC)/_build && $(MAKE)
do-install: $(PREFIX)/var/pkg/$(_NAME)
$(PREFIX)/var/pkg/$(_NAME): | do-build
	rsync -ax $(_WRKSRC)/bin/ $(PREFIX)/bin/
	rsync -ax $(_WRKSRC)/include/ $(PREFIX)/include/
	cd $(_WRKSRC) && find bin include|grep -v '^$$'|sort -r > $@
	@echo "# $(_REVISION)" >> $@
do-clean:
	$(MAKE) -C $(_WRKSRC)/_build clean || rm -rf $(_WRKSRC)/_build
do-distclean:
	cd $(_WRKSRC) && $(GIT) clean -xdf
do-flush:
	rm -rf $(_WRKSRC)
