include ./make/common.mk
include ./make/maintenance.mk

MAKEFLAGS += --no-print-directory

SPEC_FILE = run-gmp-spec.yaml

#######
# Tools
#######

TOOLS_DIR ?= $(PWD)/.tools
DISTROGEN_BIN ?= $(TOOLS_DIR)/distrogen

$(DISTROGEN_BIN):
	$(MAKE) tools-dir
	GOBIN=$(TOOLS_DIR) go install github.com/GoogleCloudPlatform/opentelemetry-operations-collector/cmd/distrogen@$(shell cat .distrogen/VERSION)

# This is a PHONY target cause if you make it as a normal recipe
# it gets very confused because the creation date of the .tools
# directory is newer than the tools inside it.
.PHONY: tools-dir
tools-dir:
	@mkdir -p $(TOOLS_DIR)

.PHONY: distrogen
distrogen: $(DISTROGEN_BIN)

##########################
# Updating OTel Components
##########################

COMPONENT_DIR = components

.PHONY: update-otel-components
update-otel-components: DISTROGEN_QUERY := $(DISTROGEN_BIN) query --spec $(SPEC_FILE) --field
update-otel-components: export OTEL_VERSION = v$(shell $(DISTROGEN_QUERY) opentelemetry_version)
update-otel-components: export OTEL_CONTRIB_VERSION = v$(shell $(DISTROGEN_QUERY) opentelemetry_contrib_version)
update-otel-components: go.work distrogen
	cd $(COMPONENT_DIR) && PATH="$(TOOLS_DIR):${PATH}" $(MAKE) update-components

.PHONY: test-otel-components
test-otel-components:
	cd $(COMPONENT_DIR) && $(MAKE) test-components

.PHONY: tidy-otel-components
tidy-otel-components:
	cd $(COMPONENT_DIR) && $(MAKE) tidy-components

###################
# Distro Generation
###################

GEN_OTEL = $(DISTROGEN_BIN) generate --spec $(SPEC_FILE) \
								 --registry ./components/registry.yaml \
								 --templates ./templates

.PHONY: gen
gen: distrogen
	@$(GEN_OTEL)

.PHONY: regen
regen: distrogen
	@$(GEN_OTEL) --force

regen-v: distrogen
	@$(GEN_OTEL) --force -v

###########
# Workspace
###########

ALL_DIRECTORIES = find . -type d  -print0
EXCLUDE_TOOLS_DIRS = grep -z -v ".*\.tools.*"
EXCLUDE_BUILD_DIRS = grep -z -v -e ".*_build.*" -e ".*dist.*"

.PHONY: workspace
workspace: go.work

go.work:
	go work init
	$(ALL_DIRECTORIES) |\
	$(EXCLUDE_TOOLS_DIRS) |\
	$(EXCLUDE_BUILD_DIRS) |\
	xargs -0 go work use

.PHONY: clean-workspace
clean-workspace:
	rm -f go.work
	rm -f go.work.sum
