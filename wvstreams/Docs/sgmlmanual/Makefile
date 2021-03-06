
default: includes wvstreams.verify

all: includes wvstreams.verify wvstreams.html wvstreams.ps wvstreams.pdf

html: wvstreams.sgml wvstreams.html
ps: wvstreams.ps
pdf: wvstreams.pdf
rtf: wvstreams.rtf

IMGFILES=$(shell [ -d img ] && find img -type f)

ALL_WVSGML=$(shell /bin/ls *.wvsgml 2>/dev/null)

%.verify: %.sgml
	# Syntax check
	nsgmls $< >/dev/null
	# Syntax check passed

%.sgml: %.wvsgml *.m4 $(ALL_WVSGML) Makefile
	@echo Preprocessing $< to $@...
	@M4PATH=egfiles:../weaver/src m4 sgmlmacros.m4 $< | awk '			\
		BEGIN {					\
			paropen = 0;			\
			parready = 0;			\
			disabled = 0;			\
			autopunc = 0;			\
		}					\
							\
 		/^[ 	]*--AUTOPUNC--[ 	]*$$/ {	\
			autopunc = !autopunc;		\
			next;				\
		}					\
							\
 		/^[ 	]*--SNIP--[ 	]*$$/ {		\
			disabled = !disabled;		\
			next;				\
		}					\
							\
							\
		(autopunc==1) {				\
			gsub("&", "\\&amp;");		\
			gsub("<", "\\&lt;");		\
			gsub(">", "\\&gt;");		\
		}					\
							\
		parready && !disabled && !paropen && /^[ 	]*[-A-Za-z0-9]/ {	\
			print "<para>";			\
			print;				\
			parready = 0;			\
			paropen = 1;			\
			next;				\
		}					\
							\
		paropen && !disabled && !parready && /^[ 	]*$$/ {	\
			print "</para>";		\
			print;				\
			parready = 1;			\
			paropen = 0;			\
			next;				\
		}					\
							\
		/^[ 	]*$$/ {				\
			parready = 1;			\
		}					\
							\
		{ print; }				\
		' | cat -s >$@

%.html: %.sgml HTML.dsl ${IMGFILES}
	${MAKE} dirimages
	rm -rf $@ $*.html.tmp
	mkdir -p $*.html.tmp
	# Copy only the required .gif files
	set -e; \
	for d in $$(grep '"[^"]*\.gif"' $*.sgml \
			| sed 's,^.*"img/\([^"]*\.gif\)".*$$,\1,'); \
	do \
		mkdir -p "$*.html.tmp/img/$$(dirname $$d)"; \
		${MAKE} img.tmp/$$d; \
		cp -v img.tmp/$$d $*.html.tmp/img/$$d; \
	done
	# Generate html
	cd $*.html.tmp && jade -t sgml -d ../HTML.dsl ../$*.sgml
	mv $*.html.tmp $@

%.rtf: %.sgml ${IMGFILES}
	$(MAKE) dirimages htmlimages
	rm -f $@
	jade -t rtf -o $*.rtf.tmp \
		-d /usr/lib/sgml/stylesheet/dsssl/docbook/nwalsh/print/docbook.dsl $<
	cat $*.rtf.tmp | sed 's,"img/\(.*\)\.[^.]*","img.tmp/\1.gif",g' >$@
	rm -f $*.rtf.tmp

%.tex: %.sgml
	rm -f $@
	jade -t tex -o $*.tex.tmp \
		-d /usr/lib/sgml/stylesheet/dsssl/docbook/nwalsh/print/docbook.dsl $<
	cat $*.tex.tmp | sed 's,{img/\(.*\)\.[^.]*},{img.tmp/\1.eps},g' >$@
	rm -f $*.tex.tmp

%.dvi: jadetex.fmt %.tex ${IMGFILES}
	${MAKE} teximages
	jadetex $*.tex </dev/null >/dev/null # generate .aux files for xrefs
	jadetex $*.tex </dev/null >/dev/null # actually generate output
	jadetex $*.tex </dev/null >/dev/null # actually generate output

%.ps: %.dvi
	dvips -q -o $@ $< </dev/null

%.pdf: %.ps
	ps2pdf $<

img.tmp:
	rm -rf img.tmp
	mkdir img.tmp

img.tmp/%: img/% img.tmp
	cp $< $@
	
img.tmp/%.eps: img.tmp/%.ps
	cp $< $@

img.tmp/%.pnm: img.tmp/%.gif
	giftopnm $< >$@

img.tmp/%.eps img.tmp/%.ps: img.tmp/%.pnm
	pnmtops -dpi 100 -width 350 $< >$@

img.tmp/%.gif: img.tmp/%.ppm
	ppmtogif -interlace $< >$@

img.tmp/%.ppm: img.tmp/%.ps
	gs -q -dNOPAUSE -sDEVICE=ppm -r75x75 -sOutputFile=$@ $< -c quit

includes: $(shell grep 'include(.*)' $(ALL_WVSGML) </dev/null | sed 's/^.*include(\(.*\)).*$$/\1/')
	
images: htmlimages teximages

dirimages:
	mkdir -p . $$([ -d img ] && find img -type d | sed 's,^img/,img.tmp/,')

# create .eps from all image types
teximages: dirimages
	${MAKE} null $$([ -d img ] && find img -type f \
			| sed 's,img/\(.*\)\.[^.]*$$,img.tmp/\1.eps,')

# create .gif from all image types			
htmlimages: dirimages
	${MAKE} null $$([ -d img ] && find img -type f \
			| sed 's,img/\(.*\)\.[^.]*$$,img.tmp/\1.gif,')

null:

jadetex.fmt:
	tex -ini -progname=jadetex \&latex jadetex.ini </dev/null >/dev/null


clean:
	$(MAKE) -C egfiles clean
	rm -f *~ *.fmt *.tex *.ps *.pdf *.dvi *.aux *.log *.rtf HTML.manifest
	rm -rf *.html *.tmp
	rm -f $$(echo "$(ALL_WVSGML)" | sed 's/\.wvsgml/.sgml/g')
