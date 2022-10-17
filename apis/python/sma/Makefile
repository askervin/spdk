#  SPDX-License-Identifier: BSD-3-Clause
#  Copyright (c) Intel Corporation.
#  All rights reserved.
#

SPDK_ROOT_DIR := $(abspath $(CURDIR)/../../..)
include $(SPDK_ROOT_DIR)/mk/spdk.common.mk

API := $(notdir $(CURDIR))

PYTHON_API_DIR := $(CURDIR)

PROTO_DIR := $(abspath $(SPDK_ROOT_DIR)/apis/proto/$(API))
PROTO_FILES = $(wildcard $(PROTO_DIR)/*.proto)

PYTHON_GEN_PBS = $(foreach proto,$(notdir $(basename $(PROTO_FILES))),$(addprefix $(PYTHON_API_DIR)/$(proto),_pb2.py _pb2_grpc.py))

all: $(PYTHON_GEN_PBS)
	$(Q)cd "$(PYTHON_API_DIR)" && \
	touch __init__.py && \
	sed -i -e 's/^\(import.*_pb2\)/from . \1/' *_pb2.py *_pb2_grpc.py

$(PYTHON_API_DIR)/%_pb2.py $(PYTHON_API_DIR)/%_pb2_grpc.py: $(PROTO_DIR)/%.proto
	$(Q)python3 -m grpc_tools.protoc --proto_path="$(PROTO_DIR)" --python_out="$(PYTHON_API_DIR)" --grpc_python_out="$(PYTHON_API_DIR)" $^

clean:
	$(Q)$(RM) $(PYTHON_GEN_PBS) "$(PYTHON_API_DIR)/__init__.py"

# TODO: we should probably write a proper install rule here instead of
# just blindly copying all python scripts when building the RPMs
install:
uninstall:

.PHONY: all clean install uninstall
