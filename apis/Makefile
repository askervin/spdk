#  SPDX-License-Identifier: BSD-3-Clause
#  Copyright (c) Intel Corporation.
#  All rights reserved.
#

SPDK_ROOT_DIR := $(abspath $(CURDIR)/..)
include $(SPDK_ROOT_DIR)/mk/spdk.common.mk

DIRS-$(CONFIG_SMA) = sma

.PHONY: $(DIRS-y)

all: $(DIRS-y)

include $(SPDK_ROOT_DIR)/mk/spdk.subdirs.mk
