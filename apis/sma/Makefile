#  SPDX-License-Identifier: BSD-3-Clause
#  Copyright (c) Intel Corporation.
#  All rights reserved.
#

SPDK_ROOT_DIR := $(abspath $(CURDIR)/../..)
include $(SPDK_ROOT_DIR)/mk/spdk.common.mk

# Generate importable go grpc interface module to "MY" fork of spdk:
#   make GO_MODULE=github.com/MY/spdk/apis/go
# If spdk.io/sma is valid go import path, generate interface to it:
#   make GO_MODULE=spdk.io
GO_MODULE ?= github.com/spdk/spdk/apis/go

PROTO_DIR := $(abspath $(CURDIR))
PROTO_FILES := $(wildcard $(SPDK_ROOT_DIR)/proto/*.proto)
PACKAGE := $(notdir $(PROTO_DIR))
MAIN_PROTO := $(PROTO_DIR)/$(PACKAGE).proto

GO_API_DIR := ../go
GO_GEN_PB := $(addprefix $(GO_API_DIR)/,$(addsuffix .pb.go,$(basename $(MAIN_PROTO))))
GO_MOD := $(GO_API_DIR)/go.mod

# Use go and protoc versions installed by pkgdep.sh
PATH := $(shell $(SPDK_ROOT_DIR)/scripts/pkgdep.path.sh 2>/dev/null || echo $$PATH)
SHELL := env PATH=$(PATH) /bin/bash

all: $(GO_MOD)

$(GO_MOD): $(GO_GEN_PB)
	$(Q)command -v go && $(RM) $@
	$(Q)cd $(GO_API_DIR) && go mod init $(GO_MODULE)
	$(Q)cd $(GO_API_DIR) && go mod tidy

$(GO_GEN_PB): $(PROTO_FILES)
	$(Q)$(RM) -r $(GO_API_DIR)/$(PACKAGE)
	$(Q)cp -v $(PROTO_FILES) .
	$(Q)sed -e "s:go_package = \"spdk.io/:go_package = \"$(GO_MODULE)/:" -i *.proto
	$(Q)mkdir -p "$(GO_API_DIR)"
	$(Q)command -v protoc; protoc --go_out="$(GO_API_DIR)" --go_opt=module="$(GO_MODULE)" --go-grpc_out="$(GO_API_DIR)" --go-grpc_opt=module="$(GO_MODULE)" *.proto

clean:
	$(Q)$(RM) *.proto

.PHONY: all clean
