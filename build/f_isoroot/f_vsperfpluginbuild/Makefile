##############################################################################
# Copyright (c) 2015 Ericsson AB and others.
# jonas.bjurel@eicsson.com
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Apache License, Version 2.0
# which accompanies this distribution, and is available at
# http://www.apache.org/licenses/LICENSE-2.0
##############################################################################

TOP := $(shell pwd)

############################################################################
# BEGIN of Include definitions
#
include config.mk
#
# END Include definitions
#############################################################################

.PHONY: all
all: .vsperfbuild

.PHONY: clean
clean:
	@rm -f .vsperfbuild ../release/opnfv/fuel-plugin-vsperf*.rpm fuel-plugin-vsperf*.rpm
	@rm -f $(BUILD_BASE)/gitinfo_vsperf-plugin.txt gitinfo_vsperf-plugin.txt

.PHONY: release
release:.vsperfbuild
	@rm -f ../release/opnfv/fuel-plugin-vsperf*.rpm
	@mkdir -p ../release/opnfv
	@cp fuel-plugin-vsperf*.rpm ../release/opnfv/
	cp gitinfo_vsperf-plugin.txt $(BUILD_BASE)
	cd $(BUILD_BASE) && mkdir -p ../deploy/templates/plugins && cp -rf $(TOP)/config/* ../deploy/templates/plugins

.vsperfbuild:
	@rm -rf fuel-plugin-vsperf
	sudo apt-get -y install build-essential ruby-dev rubygems-integration python-pip git rpm createrepo dpkg-dev
	sudo pip install fuel-plugin-builder
	../../docker/ubuntu-builder/install_docker.sh
	git clone $(VSPERF_REPO)
	cd fuel-plugin-vsperf; \
	git checkout $(VSPERF_BRANCH); \
	if [ ! -z $(VSPERF_CHANGE) ]; then \
	   git fetch $(VSPERF_REPO) $(VSPERF_CHANGE); \
	   git checkout FETCH_HEAD; \
	fi
	INCLUDE_DEPENDENCIES=true fpb --debug --build fuel-plugin-vsperf/
	@mv fuel-plugin-vsperf/fuel-plugin-vsperf*.rpm .
	$(REPOINFO) -r . > gitinfo_vsperf-plugin.txt
	@rm -rf fuel-plugin-vsperf
	@touch .vsperfbuild

#############################################################################
# Cache operations - only used when building through ci/build.sh
#############################################################################


# Create a unique hash to be used for getting and putting cache, based on:
#   - The SHA1 hash of the HEAD on the plugin repo's $(VSPERF_BRANCH)
#   - The contents of this Makefile
.cacheid:
	@if [ ! -z $(VSPERF_CHANGE) ]; then \
	  $(CACHETOOL) getcommitid $(VSPERF_REPO) $(VSPERF_CHANGE) > .cachedata; \
	else \
	  $(CACHETOOL) getcommitid $(VSPERF_REPO) $(VSPERF_BRANCH) > .cachedata; \
	fi
	@sha1sum Makefile | awk {'print $$1'} >> .cachedata
	@sha1sum config.mk | awk {'print $$1'} >> .cachedata
	@cat .cachedata | $(CACHETOOL) getid > .cacheid

# Clean local data related to caching - called prior to ordinary build
.PHONY: clean-cache
clean-cache: clean
	@rm -f .cachedata .cacheid

# Try to download cache - called prior to ordinary build
.PHONY: get-cache
get-cache: .cacheid
	@if $(CACHETOOL) check $(shell cat .cacheid); then \
		 $(CACHETOOL) get $(shell cat .cacheid) | tar xf -;\
	else \
		echo "No cache item found for $(shell cat .cacheid)" ;\
		exit 0;\
	fi

# Store cache if not already stored - called after ordinary build
.PHONY: put-cache
put-cache: .cacheid
	@tar cf - .vsperfbuild fuel-plugin-vsperf*.rpm gitinfo_vsperf-plugin.txt | $(CACHETOOL) put $(shell cat .cacheid)