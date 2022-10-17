#  SPDX-License-Identifier: BSD-3-Clause
#  Copyright (c) Intel Corporation.
#  All rights reserved.
#
SPDK_ROOT_DIR := $(abspath $(CURDIR)/../../..)
include $(SPDK_ROOT_DIR)/mk/spdk.common.mk

API := $(notdir $(CURDIR))

GO_API_DIR := $(CURDIR)

# Generate importable go grpc interface module to "MY" fork of spdk:
#   make GO_MODULE=github.com/MY/spdk/apis/go/sma
# If spdk.io/sma is valid go import path, generate interface to it:
#   make GO_MODULE=spdk.io/sma

GO_MODULE ?= github.com/spdk/spdk/apis/go/$(API)
GO_MODULE_NO_API = $(dir $(GO_MODULE))

PROTO_DIR := $(abspath $(SPDK_ROOT_DIR)/apis/proto/$(API))
PROTO_FILES := $(wildcard $(PROTO_DIR)/*.proto)

PACKAGE := $(notdir $(PROTO_DIR))

# GO_GEN_DIRS example: sma/ sma/nvme/ sma/nvmf_tcp/ ...
GO_GEN_DIRS := $(addsuffix /,$(shell awk -F'[ ;]' '/^package .*;/{print $$2}' $(PROTO_FILES) | sed 's:\.:/:g'))

# GO_PKG_DIRS example: nvme/ nvmf_tcp/ ...
GO_PKG_DIRS := $(GO_GEN_DIRS:$(API)/%=%)

# GO_GEN_PBS example: sma.pb.go nvme/nvme.pb.go nvmf_tcp/nvmf_tcp.go ...
GO_GEN_PBS := $(foreach gendir,$(GO_GEN_DIRS),$(addprefix $(gendir:$(API)/%=%),$(addsuffix .pb.go,$(notdir $(gendir:/=)))))

GO_MOD := $(GO_API_DIR)/go.mod

# Use go and protoc versions installed by pkgdep.sh
PATH := $(shell $(SPDK_ROOT_DIR)/scripts/pkgdep.path.sh 2>/dev/null || echo $$PATH)
SHELL := env PATH=$(PATH) /bin/bash

PROTOC = protoc --go_out="$(GO_API_DIR)" --go_opt=module="$(GO_MODULE)" --go-grpc_out="$(GO_API_DIR)" --go-grpc_opt=module="$(GO_MODULE)" --proto_path="$(PROTO_DIR)"

all: $(GO_PKG_DIRS) go_module_check $(GO_MOD)

help:
	@echo "Usage: make [GO_MODULE=github.com/ME/spdk/sma] all"
	@echo "       make [CLEAN_APIS=1] clean"
	@echo ""
	@echo "Defaults:"
	@echo "    GO_MODULE=$(GO_MODULE)"
	@echo "    CLEAN_APIS=$(CLEAN_APIS)"
	@echo ""
	@echo "Environment and tool versions:"
	@echo "    PATH=$$PATH"
	@echo "    go: $$(go version)"
	@echo "    protoc: $$(protoc --version)"
	@echo "    protoc-gen-go: $$(protoc-gen-go --version)"

$(GO_PKG_DIRS):
	$(Q)mkdir -p $@

go_module_check:
	$(Q)grep -q 'go_package = "$(GO_MODULE)"' $(PROTO_DIR)/$(API).proto || sed -e "s:go_package = \".*/$(API):go_package = \"$(GO_MODULE):" -i $(PROTO_DIR)/*.proto

$(GO_MOD): $(GO_GEN_PBS)
	$(Q)command -v go && $(RM) $@
	$(Q)cd $(GO_API_DIR) && go mod init $(GO_MODULE)
	$(Q)cd $(GO_API_DIR) && go mod tidy

%.pb.go: ../../../proto/$(API)/$(notdir %.proto)
	$(Q) $(PROTOC) $(abspath $<)

%.pb.go: ../../proto/$(API)/$(notdir %.proto)
	$(Q) $(PROTOC) $(abspath $<)

clean:
	@if [ "$(CLEAN_APIS)" == "1" ]; then \
	    if [ "$(Q)" != "@" ]; then set -x; fi; \
	    $(RM) *.proto $(GO_GEN_PBS) $(GO_MOD) go.sum $(API)_grpc.pb.go; \
	    for go_pkg_dir in $(GO_PKG_DIRS); do \
		rmdir "$$go_pkg_dir" 2>/dev/null || :; \
	    done; \
	else \
	    echo "Note 'make clean' keeps files in $(CURDIR) as they are in git."; \
	    echo "Run 'make CLEAN_APIS=1 clean' to remove all generated APIs."; \
	fi

# Generated go API is not to be installed on system.
# It will be used through importing from github or project sources.
# Therefore there is nothing to be done on install/uninstall.
install:
uninstall:

.PHONY: all clean install uninstall go_module_check $(GO_PKG_DIRS)
