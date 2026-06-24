#!/bin/bash
# DKMS clean script for camx camera driver.
# DKMS sets cwd to $dkms_tree/$module/$version/build/ before invoking CLEAN.
set -euo pipefail

KERNEL_VER="${1}"
BUILD_DIR="$(pwd)"
KERNEL_BUILD="/lib/modules/${KERNEL_VER}/build"

make -C "${KERNEL_BUILD}" M="${BUILD_DIR}" clean 2>/dev/null || true
rm -f "${BUILD_DIR}/cam_generated_h"
