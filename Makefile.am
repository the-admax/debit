SUBDIRS = data debian doc rpm windows altera utils xdl tests codes

if BUILD_GUI
  SUBDIRS += xiledit desktop
endif

AM_CPPFLAGS	= -DDATADIR=\"$(pkgdatadir)\"
AM_CFLAGS	= -O3

SHARED_SRC	= bitarray.c bitarray.h \
		bitheader.c bitheader.h \
		filedump.c filedump.h \
		localpips.c localpips.h \
		keyfile.c keyfile.h \
		analysis.c analysis.h \
		sites.c sites.h \
		connexity.c connexity.h \
		wiring.c wiring.h \
		bitstream_parser.h bitstream_packets.h \
		debitlog.h design.h bitstream_high.h \
		bitstream_write.c bitstream_write.h \
		xdlout.h xdlout.c

SHARED_SRC_V2	= codes/crc-ibm.c codes/crc-ibm.h \
		bitstream.c bitstream.h sites_v2.h \
		bitstream_parser.c bitstream_parser.h \
		design_v2.h virtex2_config.h cfgbit.h

bin_PROGRAMS	= debit debit_v4 debit_v5 debit_s3

if BUILD_BIT2PDF
  bin_PROGRAMS += bit2pdf
endif

debit_SOURCES	= $(SHARED_SRC) $(SHARED_SRC_V2) debit.c config.h
debit_CFLAGS	= $(AM_CFLAGS) -DVIRTEX2 @GLIB_CFLAGS@
debit_LDADD	= @GLIB_LIBS@

debit_s3_SOURCES	= $(SHARED_SRC) $(SHARED_SRC_V2) design_s3.h debit.c config.h
debit_s3_CFLAGS	= $(AM_CFLAGS) -DSPARTAN3 @GLIB_CFLAGS@
debit_s3_LDADD	= @GLIB_LIBS@

debit_v4_SOURCES	= $(SHARED_SRC) debit.c \
			bitstream_parser_common.c \
			_far_v4.h bitstream_v4.c sites_v4.h \
			design_v4.h design_common.h \
			codes/crc32-c.c codes/crc32-c.h \
			codes/xhamming.c codes/xhamming.h \
			config.h
debit_v4_CFLAGS	= $(AM_CFLAGS) -DVIRTEX4 @GLIB_CFLAGS@
debit_v4_LDADD	= @GLIB_LIBS@

debit_v5_SOURCES	= $(SHARED_SRC)	debit.c \
			bitstream_parser_common.c \
			_far_v5.h bitstream_v5.h sites_v5.h \
			design_v5.h design_common.h \
			codes/crc32-c.c codes/crc32-c.h \
			bitstream_v4.c bitstream.h \
			codes/xhamming.c codes/xhamming.h \
			config.h
debit_v5_CFLAGS	= $(AM_CFLAGS) -DVIRTEX5 @GLIB_CFLAGS@
debit_v5_LDADD	= @GLIB_LIBS@

bit2pdf_SOURCES = $(SHARED_SRC) $(SHARED_SRC_V2) bit2pdf.c sites_draw.c wiring_draw.c bitdraw.h
bit2pdf_CFLAGS = $(AM_CFLAGS) -DVIRTEX2 @GLIB_CFLAGS@ @CAIRO_CFLAGS@ @CAIRO_PDF_CFLAGS@ @CAIRO_PS_CFLAGS@ @CAIRO_SVG_CFLAGS@
bit2pdf_LDADD  = @GLIB_LIBS@ @CAIRO_LIBS@ @CAIRO_PDF_LIBS@ @CAIRO_PS_LIBS@ @CAIRO_SVG_LIBS@

#Distribution hook so that the tarballs are tagged with the git tree
#SHA1 hash
GIT_DIR		= $(srcdir)/.git
GIT_COMMIT_REF	= $(shell cat $(GIT_DIR)/refs/heads/master)
GIT_TREE_REF	= $(shell git cat-file commit $(GIT_COMMIT_REF) | head -n1 | awk '{print $$2}')

dist-hook:
	echo "$(GIT_COMMIT_REF)" > $(distdir)/git-ref

#Specific testing and profiling targets. To be reorganized

DEBIT=debit

TMPDIR=trash
datdir=$$OLDPWD/data
BITFILE_REF=$(TMPDIR)/reference.bit
REF_DIR=$(TMPDIR)/reference_dir
TEST_DIR=$(TMPDIR)/test_dir
PROFILE_DIR=$(TMPDIR)/profile_dir
DEBIT_ARGS=--pipdump --datadir=$(datdir)

checkpoint: $(DEBIT)
	mkdir -p $(REF_DIR) && pushd $(REF_DIR) && \
	$$OLDPWD/$< --input $$OLDPWD/$(BITFILE_REF) $(DEBIT_ARGS) && \
	popd

checkcheckpoint: $(DEBIT)
	mkdir -p $(TEST_DIR) && pushd $(TEST_DIR) && \
	$$OLDPWD/$< --input $$OLDPWD/$(BITFILE_REF) $(DEBIT_ARGS) && \
	popd && \
	diff -q $(REF_DIR) $(TEST_DIR) && echo "Test OK"

all-profiles: profile-mem profile-exec profile-std

# gather some profiling data. Please use !
MASSIF_ARGS=--tool=massif --depth=5 --alloc-fn=g_malloc0 --alloc-fn=g_malloc --alloc-fn=__xcalloc --alloc-fn=__xmalloc
DEBIT_MASSIF_ARGS=--datadir=$(datdir) --pipdump
profile-mem: $(DEBIT)
	mkdir -p $(PROFILE_DIR) && pushd $(PROFILE_DIR) && rm -f *.ps && \
	valgrind $(MASSIF_ARGS) $$OLDPWD/$< $(DEBIT_MASSIF_ARGS) --input $$OLDPWD/$(BITFILE_REF) &> /dev/null && \
	popd

DEBIT_EXEC_ARGS=--datadir=$(datdir) --pipdump
profile-exec: $(DEBIT)
	mkdir -p $(PROFILE_DIR) && pushd $(PROFILE_DIR) && rm -f callgrind.out.* && \
	callgrind $$OLDPWD/$< $(DEBIT_EXEC_ARGS) --input $$OLDPWD/$(BITFILE_REF) &> /dev/null && \
	callgrind_annotate callgrind.out.* --auto=yes --include=$$OLDPWD > callgrind.out && \
	popd

MEMCHECK_ARGS=--tool=memcheck --leak-check=full --leak-resolution=high --show-reachable=yes --log-file=valgrind
profile-std: $(DEBIT)
	mkdir -p $(PROFILE_DIR) && pushd $(PROFILE_DIR) && rm -f valgrind.* && \
	valgrind $(MEMCHECK_ARGS) $$OLDPWD/$< $(DEBIT_ARGS) --input $$OLDPWD/$(BITFILE_REF) &> /dev/null && \
	popd

.PHONY: doc deb srpm

# doxygen documentation generation
DOCSOURCES=$(shell find ./ -name '*.c' -o -name '*.h')
DOXYCONFIG=doc/doxygen.cfg
DOXYDIR=doc

doxy: $(DOCSOURCES) $(DOXYCONFIG)
	doxygen $(DOXYCONFIG) #&& make -C $(DOXYDIR)/latex/

DPKG_SOURCE_EXCLUDE=-i'(trash|.git|tests/.*/.*)' -Itrash -I.git -Itests/*/*

#Build both source and binary deb
deb:
	dpkg-buildpackage $(DEBBUILD) -D -rfakeroot -E -sn $(DPKG_SOURCE_EXCLUDE)

RPM_PARAMS=--clean --rmsource --sign

#Build platform-independent srpms.
#Building the actual binary rpms is left as an exercise for the reader.
srpm: dist-gzip
	TAR_OPTIONS=--wildcards rpmbuild $(RPMS_COMMON_PARAMS) -ts $(distdir).tar.gz
