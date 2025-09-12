# myAurora &nbsp; [![bluebuild build badge](https://github.com/whelanh/myaurora/actions/workflows/build.yml/badge.svg)](https://github.com/whelanh/myaurora/actions/workflows/build.yml)

See the [BlueBuild docs](https://blue-build.org/how-to/setup/) for quick setup instructions for setting up your own repository based on this template.

# My Custom Images

This repository builds two variants of a custom Aurora image:

   Standard version for machines without NVIDIA GPUs

      Image base: ghcr.io/ublue-os/aurora-dx:latest 
   
   Nvidia Version
   
      Image base: ghcr.io/ublue-os/aurora-dx-nvidia:latest

# Changes

```
Changes:
    repos:
      cleanup: true
      copr:
        - iucar/rstudio

    install:
      packages:
        - kmymoney
        - micro
        - python3-pip
        - r
        - rstudio
        - fontawesome-fonts-all
        - freetype-devel
        - fribidi-devel
        - zsh
        - java-latest-openjdk-devel
        - gcc
        - gcc-c++
        - gcc-gfortran
        - clang
        - llvm
        - sqlitebrowser
        - lftp
        - libcurl-devel
        - libjpeg-turbo-devel
        - libpng-devel
        - libtiff-devel
        - llvm-devel
        - meson
        - micro
        - tcl8-devel
        - tk8-devel
        - typescript
        - openrgb

  - type: default-flatpaks
    configurations:
      - install:
          - io.github.benini.scid
          - io.github.shiftey.Desktop  # github desktop
          - io.github.dvlv.boxbuddyrs
          - be.alexandervanhee.gradia
          - com.github.xournalpp.xournalpp

  #- type: gnome-extensions
  #  install:
  #    - Forge

  # Chezmoi setup
  - from-file: custom/chezmoi.yml
```

## Installation

> [!WARNING]  
> [This is an experimental feature](https://www.fedoraproject.org/wiki/Changes/OstreeNativeContainerStable), try at your own discretion.

To rebase an existing atomic Fedora installation to the latest build:

- First rebase to the unsigned image, to get the proper signing keys and policies installed:
  ```
  rpm-ostree rebase ostree-unverified-registry:ghcr.io/whelanh/myaurora-nvidia:latest
  ```
  or for non-NVIDIA
    ```
  rpm-ostree rebase ostree-unverified-registry:ghcr.io/whelanh/myaurora-amd:latest
  ```

- Reboot to complete the rebase:
  ```
  systemctl reboot
  ```
- Then rebase to the signed image, like so:
  ```
  rpm-ostree rebase ostree-image-signed:docker://ghcr.io/whelanh/myaurora-nvidia:latest
  ```
  or for non-NVIDIA
  ```
  rpm-ostree rebase ostree-image-signed:docker://ghcr.io/whelanh/myaurora-amd:latest
  ```
- Reboot again to complete the installation
  ```
  systemctl reboot
  ```

The `latest` tag will automatically point to the latest build. That build will still always use the Fedora version specified in `recipe.yml` (which is Aurora:latest).

## ISO

If build on Fedora Atomic, you can generate an offline ISO with the instructions available [here](https://blue-build.org/learn/universal-blue/#fresh-install-from-an-iso). These ISOs cannot unfortunately be distributed on GitHub for free due to large sizes, so for public projects something else has to be used for hosting.

## Verification

These images are signed with [Sigstore](https://www.sigstore.dev/)'s [cosign](https://github.com/sigstore/cosign). You can verify the signature by downloading the `cosign.pub` file from this repo and running the following command:

```bash
cosign verify --key cosign.pub ghcr.io/whelanh/myaurora-amd:latest
```
