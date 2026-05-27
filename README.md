# CAMX (Camera X) Kernel Driver

This repository contains the source code of the Camera downstream driver.
Required to run the camera on Qualcomm Snapdragon targets.

## Overview

CAMX driver is responsible for:

- Camera sensor management and configuration
- Image Signal Processor (ISP/Spectra) pipeline control
- Camera Request Manager (CRM) for multi-camera synchronization
- Memory management for camera buffers (ION/DMA)
- Power management and bandwidth voting for camera subsystems
- Camera hardware register programming (IFE, BPS, IPE, ICP, JPEG, CCI)
- Providing V4L2/media controller interface to userspace
- Sensor communication via I2C/CCI (Camera Control Interface)
- Flash, actuator, and OIS (Optical Image Stabilization) sub-device control

## Branches

Primary development branch: `camera-kernel.qclinux.0.0`

## How-to-Build

This source code can be built using the recipe file related to the camx
driver in the meta-qcom project, which is designed for Qualcomm-based
platforms.

### Steps for compilation:

1. Modify the camera driver code as required and generate a corresponding
   patch file.
2. Git clone the meta-qcom project:
   - git clone https://github.com/qualcomm-linux/meta-qcom.Git
3. Integrate the patch into the camx-dlkm related recipe file located at:
`meta-qcom/recipes-multimedia/camx`
4. Compile the project. Please refer to the `README.md` file provided in
the meta-qcom repository.

## Getting in Contact

Problems specific to the camera driver can be reported in the **Issues**
section of this repository.

## License

This driver is released under the **GPL-2.0** license.
See `LICENSE.txt` for details.
