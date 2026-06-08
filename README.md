# Camera Kernel Driver

This repository contains the Qualcomm Spectra camera kernel driver (DLKM).
Required to run the camera subsystem on Qualcomm QCS/Snapdragon platforms.

## Overview

The camera driver is responsible for:

- Camera sensor management via I2C/CCI (Camera Control Interface)
- Image Signal Processor (ISP/Spectra) pipeline control — IFE, CSID, SFE, VFE
- Image compute offload — ICP firmware, BPS, IPE, OFE
- Camera Request Manager (CRM) for multi-camera pipeline synchronization
- CPAS: clock, AXI bandwidth, and power management for all camera hardware
- Memory management for camera DMA buffers (SMMU/IOMMU)
- JPEG hardware encode and DMA
- Sub-device control: actuator, OIS, flash, EEPROM, CSIPHY
- V4L2/media-controller interface to userspace

## Driver Stacks

The repository contains two camera driver stacks to support different chipset
generations without mixing hardware-specific code:

| Directory    | Chipset                                     | Generation                        |
|--------------|---------------------------------------------|-----------------------------------|
| `camera_kt/` | QCM6490 (qcs6490)                           | Older — A5 ICP, VFE17x, TFE, OPE |
| `camera/`    | QCS615, QCS9100, QCS8300, QCS8550, X1E80100 | Current — ICP v1/v2, SFE, CRE    |

The `common/` directory holds utilities shared by both stacks. If a new
chipset generation requires a distinct hardware stack, a third directory
would be added alongside these two.

## Kernel Modules

A single build produces **one `.ko` per supported architecture**. The
top-level `Makefile` defines:

```
SUPPORTED_ARCH = qcm6490 qcs9100 qcs615 x1e80100
```

Each architecture maps to a named module:

| Architecture | Kernel Module        | Stack used   |
|--------------|----------------------|--------------|
| `qcm6490`    | `camera_qcm6490.ko`  | `camera_kt/` |
| `qcs615`     | `camera_qcs615.ko`   | `camera/`    |
| `qcs9100`    | `camera_qcs9100.ko`  | `camera/`    |
| `x1e80100`   | `camera_x1e80100.ko` | `camera/`    |

Per-arch feature flags live in `config/<arch>-camera.mk` and control which
subsystems (ISP, ICP, JPEG, sensor, FD, LRME, CRE …) are compiled into
that module. All modules are installed to `/lib/modules/<kver>/updates/camera/`.

## Module Selection at Runtime

At boot the kernel loads the correct `.ko` by matching the `compatible` string
in the board's Device Tree against the driver's `of_match_table`.

Each board's `*-camera.dtsi` must contain a `qcom,camera-main` node:

```dts
/* Example: X1E80100 platform (glymur-camera.dtsi) */
qcom,camera-main {
    compatible = "qcom,camera_x1e80100";
    status = "okay";
};

/* Example: QCS6490 platform (qcs6490-camera.dtsi) */
qcom,camera-main {
    compatible = "qcom,camera_kt";
    status = "okay";
};
```

The `compatible` string selects which `.ko` binds to the device. All other
modules present on the filesystem are ignored by the kernel's driver-match
logic. Only the matching module initialises the camera subsystems and creates
the `/dev/media*` and `/dev/video*` nodes.

## Branches

Primary development branch: `camera-kernel.qclinux.0.0`
Debian packaging branch: `debian/latest`

## How-to-Build

This source code can be built using the recipe file related to the camera
driver in the meta-qcom project, which is designed for Qualcomm-based
platforms.

### Steps for compilation:

1. Modify the camera driver code as required and generate a corresponding
   patch file.
2. Git clone the meta-qcom project:
   - `git clone https://github.com/qualcomm-linux/meta-qcom.git`
3. Integrate the patch into the camera-dlkm related recipe file located at:
   `meta-qcom/recipes-multimedia/camera-kernel`
4. Compile the project. Please refer to the `README.md` provided in the
   meta-qcom repository.

## Debian Package

The camera driver is also packaged as a DKMS Debian package hosted at:

> **https://github.com/qualcomm-linux/pkg-camx-dkms**

DKMS automatically builds and installs the correct camera kernel module(s)
for the running kernel, and rebuilds them on every kernel update.

### Download a Pre-built Package

Pre-built packages are produced by the CI pipeline. To download:

1. Go to the Actions page:
   **https://github.com/qualcomm-linux/pkg-camx-dkms/actions/workflows/build-debian-package**
2. Select the latest successful workflow run.
3. Download the `.deb` artifact from the **Artifacts** section at the bottom
   of the run page.

Two packages are produced per build:

| Package         | Install on device? | Description                              |
|-----------------|--------------------|------------------------------------------|
| `camx-dkms`     | Yes                | Camera kernel modules (DKMS);            |
|                 |                    | auto-rebuilds on kernel updates          |
| `camx-uapi-dev` | No — build host    | UAPI headers; defines the kernel–        |
|                 |                    | userspace ABI for camera components      |

### Install on Device

Install only `camx-dkms` on the target device:

```bash
sudo dpkg -i camx-dkms_<version>_arm64.deb
```

### Using `camx-uapi-dev` on a Build Host

`camx-uapi-dev` is **not** intended for installation on the target device.
Install it on the **build host** when compiling camera userspace components
(e.g. CAMX, camera HAL) that need the camera kernel UAPI headers:

```bash
sudo dpkg -i camx-uapi-dev_<version>_arm64.deb
```

Headers are installed to `/usr/include/camx/` and provide the
kernel–userspace interface definitions required to maintain ABI compatibility
between the camera kernel driver and the camera userspace stack.

### Requirements

- Architecture: `arm64`
- `dkms >= 2.2`
- `linux-headers` matching the running kernel (`linux-headers-generic` recommended)
- Kernel built with `CONFIG_ARCH_QCOM=y`
- Kernel version `>= 6.16`

### Build the Package Locally

To trigger a package build via the CI workflow:

1. Go to the Actions page:
   **https://github.com/qualcomm-linux/pkg-camx-dkms/actions/workflows/build-debian-package**
2. Click **Run workflow**.
3. Set `debian-ref` to the branch or tag to build (e.g. `debian/qcom-next`).
4. Set `suite` to the target distribution (e.g. `noble`, `trixie`, `unstable`).

To build locally using standard Debian tooling:

```bash
git clone https://github.com/qualcomm-linux/pkg-camx-dkms.git
cd pkg-camx-dkms
git checkout debian/latest
gbp buildpackage --git-ignore-branch
```

The Debian packaging lives in the `debian/` directory. The `debian/rules`
file stages the full camera-kernel source tree into the DKMS source directory
(`/usr/src/camx-<version>/`) and generates the per-arch `dkms.conf` entries
for all supported targets at package build time.

---


## Building and Flashing a New DTB

When camera Device Tree changes are made (e.g. adding a new sensor, updating
a `compatible` string), a new DTB must be compiled and flashed to the device.

### Prerequisites

```bash
mkdir kernel_deb && cd kernel_deb

# Fetch build scripts
git clone --depth=1 https://github.com/qualcomm-linux/qcom-build-utils.git
```

### Build the Kernel and DTB

Sync the kernel source (update the branch tip to the latest before cloning):

```bash
git clone --depth=1 https://github.com/qualcomm-linux/kernel.git \
    -b qcom-next-7.0-rc2-20260317 qcom-next
```

Build the kernel and produce a `.deb` package:

```bash
export CROSS_COMPILE=aarch64-linux-gnu-
export BUILD_TOP=$(pwd)
mkdir -p ${BUILD_TOP}/out

./qcom-build-utils/kernel/scripts/enable_squashfs_configs.sh qcom-next/
./qcom-build-utils/kernel/scripts/build_kernel.sh
sudo ./qcom-build-utils/kernel/scripts/build-kernel-deb.sh out/
```

### Create the Combined DTB (`dtb.bin`)

Create a manifest listing the target DTB (replace with your board's DTB name):

```bash
echo "hamoa-iot-evk.dtb" > dtb-manifest
```

Build the combined DTB image:

```bash
sudo ./qcom-build-utils/kernel/scripts/build-dtb-image.sh \
    -dtb-src ./qcom-next/arch/arm64/boot/dts/qcom \
    -manifest ./dtb-manifest \
    -out dtb.bin
```

Alternatively, build a FIT DTB image directly from the kernel `.deb`:

```bash
sudo ./qcom-build-utils/kernel/scripts/build-dtb-image.sh \
    --fit-image \
    --kernel-deb linux-kernel-<version>-arm64.deb \
    --out dtb.bin
```

### Flash `dtb.bin` via PCAT

Use PCAT to sideload the new `dtb.bin` without a full reflash:

1. Load a META build in PCAT and connect the device in **EDL (9008) mode**.
2. Set **Memory Type** to `MEMORY_TYPE_SPINOR`.
3. Open **Partition Manager**.
4. Select the **`dtb_a`** partition and click **Load**.
5. Select your `dtb.bin` file and confirm.
6. Reboot the device — the new DTB takes effect immediately.

---


---

## Camera Probe Validation

After flashing `dtb.bin` and rebooting the device, verify that the camera
driver has probed successfully.

### Reboot

```bash
sudo reboot
```

### Confirm the Correct Module is Loaded

```bash
lsmod | grep camera
```

Expected output (example for an X1E80100 platform):

```
camera_x1e80100    1234567  0
```

Only the module whose `compatible` string matches the board's DT node will
appear. If the module is missing, check `dmesg` for probe errors:

```bash
dmesg | grep -i "camera\|cam_"
```

### Confirm Camera Nodes are Present

```bash
ls /dev/media* /dev/video*
```

A successful probe creates at least one `/dev/media*` node (media controller)
and one or more `/dev/video*` nodes (V4L2 sub-devices).

---

## Getting in Contact

Problems specific to the camera driver can be reported in the **Issues**
section of this repository.

## License

This driver is released under the **GPL-2.0-only** license.
See `LICENSE.txt` for details.
