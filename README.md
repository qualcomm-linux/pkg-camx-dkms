# pkg-camx-dkms

This repository contains the Debian packaging rules and scripts for the
CAMX (Camera X) kernel driver, the downstream camera driver required to
run the camera on Qualcomm Snapdragon targets. The upstream source lives
in [qualcomm-linux/camera-driver](https://github.com/qualcomm-linux/camera-driver).

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

The driver is delivered as a DKMS module so it is rebuilt against the
running kernel on the target.

## Branches

- **qli-ci**: The primary branch containing workflow logic in the
  `.github/` folder, along with boilerplate documentation files such as
  license, contribution guidelines, and this README.
- **qcom/debian/latest**: Qualcomm's tip of Debian packaging development
  for camx-dkms.
- **qcom/debian/trixie**: Qualcomm's Debian Trixie packaging branch for
  camx-dkms.
- **qcom/ubuntu/resolute**: Qualcomm's packaging branch targeting Ubuntu
  26.04 (Resolute Raccoon).

## Typical Workflows

1. **Upstream promotion**: When the upstream camera-driver project tags
   a new release, the promote workflow merges it into the packaging
   branch and opens a PR.
2. **PR validation**: PRs in this repo are validated against the
   package build to catch breakages early.
3. **Release**: A manual dispatch finalizes the changelog, builds the
   package, uploads artifacts, and notifies
   [qcom-distro-images](https://github.com/qualcomm-linux/qcom-distro-images).

The workflows of this repo use the reusable workflows from
[qcom-build-utils](https://github.com/qualcomm-linux/qcom-build-utils)
in the background.

## Installation

```sh
sudo dpkg -i camx-dkms_<version>_arm64.deb
```

## Getting in Contact

* [Report an Issue on GitHub](../../issues)
* [Open a Discussion on GitHub](../../discussions)
* [E-mail us](mailto:Maintainers.pkg-camx-dkms@qualcomm.com) for general
  questions

## License

pkg-camx-dkms is licensed under the GPL-2.0 license, matching the
upstream camx driver. See [LICENSE.txt](LICENSE.txt) for the full
license text.
