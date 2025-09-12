# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

myAurora is a BlueBuild-based custom OS image repository that builds two variants of a personal Aurora Linux distribution:
- **Standard (AMD)**: Based on `ghcr.io/ublue-os/aurora-dx` for non-NVIDIA machines  
- **NVIDIA**: Based on `ghcr.io/ublue-os/aurora-dx-nvidia` for NVIDIA GPU machines

The project uses BlueBuild's declarative YAML configuration system to customize base Universal Blue images with additional packages, Flatpaks, GNOME extensions, and dotfiles integration via chezmoi.

## Build System and Commands

### Primary Build Commands
```bash
# Trigger manual build via GitHub Actions
gh workflow run build.yml

# View build status
gh workflow list
gh run list --workflow=build.yml

# Test recipe syntax locally (requires bluebuild CLI)
bluebuild template recipes/recipe-standard.yml
bluebuild template recipes/recipe-nvidia.yml
```

### Local Development and Testing
```bash
# Validate YAML syntax
yamllint recipes/recipe-*.yml

# Check GitHub Actions workflow syntax
actionlint .github/workflows/build.yml

# Verify cosign public key
cosign verify --key cosign.pub ghcr.io/whelanh/myaurora-amd:latest
```

### Container Image Commands
```bash
# Rebase to unsigned image (first time)
rpm-ostree rebase ostree-unverified-registry:ghcr.io/whelanh/myaurora-amd:latest
# OR for NVIDIA:
rpm-ostree rebase ostree-unverified-registry:ghcr.io/whelanh/myaurora-nvidia:latest

# Rebase to signed image (after first reboot)
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/whelanh/myaurora-amd:latest
# OR for NVIDIA:
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/whelanh/myaurora-nvidia:latest
```

## Architecture and Structure

### Repository Structure
```
/
├── recipes/                    # BlueBuild recipe configurations
│   ├── recipe-standard.yml    # AMD/non-NVIDIA variant config
│   ├── recipe-nvidia.yml      # NVIDIA variant config  
│   └── custom/
│       └── chezmoi.yml        # Dotfiles integration config
├── files/                     # Files to copy into image
│   ├── system/                # System-level file overlays
│   └── scripts/               # Custom scripts
├── modules/                   # Custom BlueBuild modules (empty)
└── .github/workflows/         # CI/CD automation
    └── build.yml              # Daily builds + PR testing
```

### BlueBuild Module Pipeline
Recipes execute modules in sequential order:

1. **files**: Copy system files from `files/system/` to image root
2. **dnf**: Install RPM packages from Fedora repos + COPR repos
3. **default-flatpaks**: Install Flatpak applications  
4. **gnome-extensions**: Install GNOME Shell extensions
5. **rpm-ostree**: Install external RPM packages from URLs
6. **chezmoi**: Apply dotfiles from external repository
7. **signing**: Configure image signing for security

### Key Configuration Differences

**Standard vs NVIDIA variants:**
- Base image: `aurora-dx` vs `aurora-dx-nvidia`
- Both variants currently have identical package sets and configurations
- Both variants include the same Flatpak applications
- GNOME extensions are currently commented out in both variants

### External Dependencies
- **Base Images**: Universal Blue project images (ghcr.io/ublue-os/*)
- **COPR Repository**: `iucar/rstudio` for RStudio packages
- **Dotfiles**: Personal dotfiles repo at `https://github.com/whelanh/dotfiles`
- **Container Registry**: GitHub Container Registry (ghcr.io)
- **Signing**: Sigstore/cosign for image verification

## Development Workflow

### Modifying Recipes
- Edit `recipes/recipe-standard.yml` or `recipes/recipe-nvidia.yml`
- Changes automatically trigger builds on push (excluding `*.md` files)
- Both variants share most configuration; sync changes between them

### Adding New Packages
- **RPM packages**: Add to `modules[].install.packages` list in dnf module
- **Flatpaks**: Add to `modules[].configurations[].install` in default-flatpaks module  
- **GNOME extensions**: Add to `modules[].install` in gnome-extensions module

### Testing Changes
- Push changes to trigger automatic build
- Use `workflow_dispatch` for manual builds
- Builds run daily at 06:00 UTC to stay current with base images

### File System Modifications  
- Add files to `files/system/` following desired final path structure
- Files automatically copied to image root during build
- Use for configuration files, systemd units, etc.

## Build Automation

### GitHub Actions Workflow
- **Triggers**: Daily schedule, push to main, pull requests, manual dispatch
- **Matrix Strategy**: Builds both recipe variants (currently only standard enabled)
- **Security**: Uses cosign for image signing with repository secrets
- **Registry**: Publishes to GitHub Container Registry with automatic cleanup

### Concurrent Builds
- Only one build runs at a time per branch
- Previous builds cancelled when new commits pushed
- Fail-fast disabled for matrix builds

The project follows immutable infrastructure principles - changes are made by rebuilding entire container images rather than modifying running systems.