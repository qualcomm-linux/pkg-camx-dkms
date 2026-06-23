#!/bin/bash
# DKMS build script for camx camera driver.
# DKMS sets cwd to $dkms_tree/$module/$version/build/ before invoking MAKE.
# M=$(pwd) and CAMERA_KERNEL_ROOT=$(pwd) ensure .ko files and Kbuild
# include paths resolve correctly from the build directory.
set -euo pipefail

KERNEL_VER="${1}"
BUILD_DIR="$(pwd)"
KERNEL_BUILD="/lib/modules/${KERNEL_VER}/build"

{
    echo "#define CAMERA_COMPILE_TIME \"$(date)\""
    echo "#define CAMERA_COMPILE_HOST \"$(uname -n)\""
    CC_VER=$(LC_ALL=C gcc --version 2>/dev/null | head -n1)
    echo "#define CAMERA_CC_VERSION \"${CC_VER}\""
} > "${BUILD_DIR}/cam_generated_h"

SUPPORTED_ARCH=$(grep "^SUPPORTED_ARCH" "${BUILD_DIR}/Makefile" \
    | sed 's/SUPPORTED_ARCH[[:space:]]*=[[:space:]]*//')

if [ -z "${SUPPORTED_ARCH}" ]; then
    echo "ERROR: SUPPORTED_ARCH not found in ${BUILD_DIR}/Makefile" >&2
    exit 1
fi

BUILD_FAILED=0
for ARCH in ${SUPPORTED_ARCH}; do
    if make -j`nproc` -C "${KERNEL_BUILD}" \
            M="${BUILD_DIR}" \
            CAMERA_KERNEL_ROOT="${BUILD_DIR}" \
            CAMERA_ARCH="${ARCH}" \
            INSTALL_MOD_DIR=camera \
            modules; then
        echo "camera_${ARCH}.ko built OK"
    else
        echo "ERROR: build failed for CAMERA_ARCH=${ARCH}" >&2
        BUILD_FAILED=1
    fi
done

[ "${BUILD_FAILED}" -eq 0 ] || exit 1
