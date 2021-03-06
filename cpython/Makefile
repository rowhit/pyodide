PYODIDE_ROOT=$(abspath ..)
include ../Makefile.envs

ROOT=$(abspath .)

HOSTINSTALL=$(ROOT)/build/$(PYVERSION)/host
HOSTBUILD=$(HOSTINSTALL)/Python-$(PYVERSION)
HOSTPYTHON=$(HOSTINSTALL)/bin/python3$(EXE)
HOSTPGEN=$(HOSTINSTALL)/bin/pgen$(EXE)

BUILD=$(ROOT)/build/$(PYVERSION)/Python-$(PYVERSION)
INSTALL=$(ROOT)/installs/python-$(PYVERSION)
TARBALL=$(ROOT)/downloads/Python-$(PYVERSION).tgz
URL=https://www.python.org/ftp/python/$(PYVERSION)/Python-$(PYVERSION).tgz
LIB=libpython$(PYMINOR).a


ZLIBVERSION = 1.2.11
ZLIBTARBALL=$(ROOT)/downloads/zlib-$(ZLIBVERSION).tar.gz
ZLIBBUILD=$(ROOT)/build/zlib-$(ZLIBVERSION)
ZLIBURL=https://zlib.net/zlib-1.2.11.tar.gz


all: $(INSTALL)/lib/$(LIB)


$(INSTALL)/lib/$(LIB): $(BUILD)/$(LIB)
	( \
		cd $(BUILD); \
		sed -i -e 's/libinstall:.*/libinstall:/' Makefile; \
		touch $(BUILD)/$(LIB) ; \
		emmake make HOSTPYTHON=$(HOSTPYTHON) PYTHON_FOR_BUILD=$(HOSTPYTHON) CROSS_COMPILE=yes inclinstall libinstall $(LIB) && \
		cp $(LIB) $(INSTALL)/lib/ && \
		cp $(HOSTINSTALL)/lib/python$(PYMINOR)/`$(HOSTPYTHON) -c "import sysconfig; print(sysconfig._get_sysconfigdata_name())"`.py $(INSTALL)/lib/python$(PYMINOR)/_sysconfigdata__emscripten_.py; \
		sed -i -e 's#'"$(PYODIDE_ROOT)"'##g' $(INSTALL)/lib/python$(PYMINOR)/_sysconfigdata__emscripten_.py; \
	)


clean:
	-rm -fr $(HOSTINSTALL)
	-rm -fr $(BUILD)
	-rm -fr $(INSTALL)


$(TARBALL):
	[ -d $(ROOT)/downloads ] || mkdir $(ROOT)/downloads
	wget -q -O $@ $(URL)
	md5sum --quiet --check checksums || (rm $@; false)


$(ZLIBTARBALL):
	[ -d $(ROOT)/downloads ] || mkdir $(ROOT)/downloads
	wget -q -O $@ $(ZLIBURL)


$(HOSTPYTHON) $(HOSTPGEN): $(TARBALL)
	mkdir -p $(HOSTINSTALL)
	[ -d $(HOSTBUILD) ] || tar -C $(HOSTINSTALL) -xf $(TARBALL)
	( \
		cd $(HOSTBUILD); \
		./configure --prefix=$(HOSTINSTALL) || cat config.log && \
	  make regen-grammar && \
		make install && \
		cp Parser/pgen$(EXE) $(HOSTINSTALL)/bin/ && \
		make distclean \
	)


$(BUILD)/.patched: $(TARBALL)
	[ -d $(BUILD) ] || (mkdir -p $(dir $(BUILD)); tar -C $(dir $(BUILD)) -xf $(TARBALL))
	cat patches/*.patch | (cd $(BUILD) ; patch -p1)
	touch $@


$(ZLIBBUILD)/.patched: $(ZLIBTARBALL)
	[ -d $(ROOT)/build ] || (mkdir $(ROOT)/build)
	tar -C $(ROOT)/build/ -xf $(ROOT)/downloads/zlib-1.2.11.tar.gz
	cat patches/zlib/*.patch | (cd $(ZLIBBUILD) ; patch -p1)
	touch $@


$(BUILD)/Makefile: $(BUILD)/.patched $(ZLIBBUILD)/.patched
	cp config.site $(BUILD)/
	( \
		cd $(BUILD); \
		CONFIG_SITE=./config.site READELF=true emconfigure ./configure --without-pymalloc --disable-shared --disable-ipv6 --without-gcc --host=asmjs-unknown-emscripten --build=$(shell $(BUILD)/config.guess) --prefix=$(INSTALL) ; \
	)


$(BUILD)/$(LIB): $(BUILD)/Makefile $(HOSTPYTHON) $(HOSTPGEN) Setup.local
	cp Setup.local $(BUILD)/Modules/
	( \
		cp build/$(PYVERSION)/host/lib/python$(PYMINOR)/`$(HOSTPYTHON) -c "import sysconfig; print(sysconfig._get_sysconfigdata_name())"`.py build/$(PYVERSION)/Python-$(PYVERSION)/Lib/_sysconfigdata__emscripten_.py; \
		cd $(BUILD); \
		emmake make HOSTPYTHON=$(HOSTPYTHON) HOSTPGEN=$(HOSTPGEN) CROSS_COMPILE=yes $(LIB) \
	)
	touch $(BUILD)/$(LIB)
