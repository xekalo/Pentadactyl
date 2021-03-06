#### configuration

TOP           = $(shell pwd)
OS            = $(shell uname -s)
BUILD_DATE    = $(shell date "+%Y/%m/%d %H:%M:%S")
BASE          = $(TOP)/../common
GOOGLE_PROJ   = dactyl
GOOGLE	      = https://$(GOOGLE_PROJ).googlecode.com/files
VERSION	     ?= $(shell sed -n 's/.*em:version\(>\|="\)\(.*\)["<].*/\2/p' $(TOP)/install.rdf | sed 1q)
UUID   	      = $(shell sed -n 's/.*em:id\(>\|="\)\(.*\)["<].*/\2/p' $(TOP)/install.rdf | sed 1q)

LOCALEDIR     = locale
DOC_FILES     = $(wildcard $(LOCALEDIR)/*/*.xml)

export VERSION BUILD_DATE
MAKE_JAR      = sh $(BASE)/make_jar.sh

# TODO: specify source files manually?
JAR_BASES     = $(TOP) $(BASE)
JAR_DIRS      = content skin locale
JAR_TEXTS     = js css dtd xml xul html xhtml xsl
JAR_BINS      = png

JAR           = chrome/$(NAME).jar

XPI_BASES     = $(JAR_BASES) $(TOP)/..
XPI_FILES     = install.rdf TODO AUTHORS Donors NEWS LICENSE.txt
XPI_DIRS      = modules components chrome defaults
XPI_TEXTS     = js jsm
XPI_BINS      = jar

XPI_NAME      = $(NAME)_$(VERSION)
XPI_PATH      = ../downloads/$(XPI_NAME)
XPI           = $(XPI_PATH).xpi

RDF           = ../downloads/update.rdf
RDF_IN        = $(RDF).in

BUILD_DIR     = build.$(VERSION).$(OS)

AWK       ?= awk
B64ENCODE ?= base64
CURL      ?= curl

.SILENT:

#### rules

TARGETS = all help info jar xpi install clean distclean install installxpi $(JAR)
$(TARGETS:%=\%.%):
	echo MAKE $* $(@:$*.%=%)
	$(MAKE) -C $* $(@:$*.%=%)

.PHONY: $(TARGETS)
all: help

help:
	@echo "$(NAME) $(VERSION) build"
	@echo
	@echo "  make help       - display this help"
	@echo "  make info       - show some info about the system"
	@echo "  make jar        - build a JAR ($(JAR))"
	@echo "  make xpi        - build an XPI ($(XPI_NAME))"
	@echo "  make installxpi - build an XPI and install it to your profile"
	@echo "  make install    - installs this source tree directly to your Firefox profile"
	@echo '                    set $$PROFILE to select a profile and $$PROFILEPATHS to change'
	@echo '                    the directory where profiles are searched'
	@echo "  make release    - updates update.rdf (this is not for you)"
	@echo "  make dist       - uploads to Google Code (this is not for you)"
	@echo "  make clean      - clean up"
	@echo "  make distclean  - clean up more"
	@echo
	@echo "running some commands with V=1 will show more build details"

info:
	@echo "version             $(VERSION)"
	@echo "release file        $(XPI)"
	@echo "doc files           $(DOC_FILES)"
	@echo "xpi files           $(XPI_FILES)"

xpi: $(XPI)
jar: $(JAR)

release: $(XPI) $(RDF)

# This is not for you!
dist: $(XPI)
	@echo DIST $(XPI) $(GOOGLE)
	set -e;							        	\
								        	\
	proj=$$(echo -n $(NAME) | sed 's/\(.\).*/\1/' | tr a-z A-Z);    	\
	proj="$$proj$$(echo $(NAME) | sed 's/.//')";                    	\
	[ -z "$$summary" ] && summary="$$proj $(VERSION) Release"; 		\
	labels="Project-$$proj,$(labels)";					\
	[ -n "$(featured)" ] && labels="$$labels,Featured";             	\
								        	\
	IFS=,; for l in $$labels; do						\
		set -- "$$@" --form-string "label=$$l";				\
	done;									\
	auth=$$(echo -n "$(GOOGLE_USER):$(GOOGLE_PASS)" | $(B64ENCODE));	\
	$(CURL) "$$@" --form-string "summary=$$summary" 			\
		-F "filename=@$(XPI)"						\
		-H "Authorization: Basic $$auth"				\
		-i "$(GOOGLE)" | sed -n '/^Location/{p;q;}'

install:
	export dir; 								\
	for dir in $(PROFILEPATHS); do 						\
		test -f "$$dir/profiles.ini" && break; 				\
	done; 									\
										\
	profile=$$(sed 's/^$$/\#/' "$$dir/profiles.ini" |			\
		awk -v"profile=$(PROFILE)" 					\
			'BEGIN { RS="#" } 					\
			index($$0, "\nName=" profile "\n") { print; exit } 	\
			!profile && /\nName=default\n/ { args["name=default"] = $$0 } 	\
			!profile && /\nDefault=1/ { args["default=1"] = $$0 } 	\
			END { if (args["default=1"]) print args["default=1"]; else print args["name=default"] }' |\
		awk -F= '{ args[$$1] = $$2 } 					\
		END {\
			if (args["IsRelative"]) print ENVIRON["dir"] "/" args["Path"];	\
			else print args["Path"]					\
		}'); 								\
										\
	if ! test -d "$$profile"; then 						\
		echo >&2 "Can't locate profile directory"; 			\
		exit 1; 							\
	fi; 									\
										\
	ext="$$profile/extensions/$(UUID)"; 					\
	rm -rf "$$ext.xpi" "$$ext"; 						\
	echo "Installing to $$ext"; 						\
	if which cygpath >/dev/null 2>&1; 					\
	then cygpath -wa .; 							\
	else pwd; 								\
	fi >"$$ext"
installxpi: xpi
	$(FIREFOX) $(XPI)

$(RDF): $(RDF_IN) Makefile
	@echo "Preparing release..."
	$(SED) -e "s,@VERSION@,$(VERSION),g" \
	           -e "s,@DATE@,$(BUILD_DATE),g" \
	           < $< > $@
	@echo "SUCCESS: $@"

clean:
	@echo "General $(NAME) cleanup..."
	rm -f $(JAR) $(XPI)

distclean:
	@echo "More $(NAME) cleanup..."
	rm -rf $(BUILD_DIR)

#### xpi

$(XPI): $(JAR)
	@echo "Building XPI..."
	mkdir -p $(XPI_PATH)
	$(AWK) -v 'name=$(NAME)' -f $(BASE)/process_manifest.awk $(TOP)/chrome.manifest >$(XPI_PATH)/chrome.manifest
	$(MAKE_JAR) "$(XPI)" "$(XPI_BASES)" "$(XPI_DIRS)" "$(XPI_TEXTS)" "$(XPI_BINS)" "$(XPI_FILES)"
	@echo "Built XPI: $@"

#### jar

$(JAR):
	@echo "Building JAR..."
	$(MAKE_JAR) "$(JAR)" "$(JAR_BASES)" "$(JAR_DIRS)" "$(JAR_TEXTS)" "$(JAR_BINS)" "$(JAR_FILES)"
	@echo "SUCCESS: $@"

