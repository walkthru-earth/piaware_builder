# PiAware Builder - walkthru-earth Fork

> **Fork of [flightaware/piaware_builder](https://github.com/flightaware/piaware_builder)** with added support for Debian 13 (Trixie) and automated CI/CD builds for multiple architectures.

## What's Different in This Fork?

| Feature | Upstream | This Fork |
|---------|----------|-----------|
| **Debian 13 (Trixie)** | ❌ Not supported | ✅ Full support |
| **Automated Builds** | Jenkins (internal) | ✅ GitHub Actions |
| **Pre-built Releases** | Not public | ✅ GitHub Releases |
| **armhf (Pi Zero W)** | Manual build | ✅ Automated |
| **arm64 (Pi 4/5)** | Manual build | ✅ Automated |
| **amd64 (x86_64)** | Manual build | ✅ Automated |

## Supported Platforms

| Debian Version | Codename | Python | cx_Freeze | Status |
|----------------|----------|--------|-----------|--------|
| 13 | Trixie | 3.12 | 7.2.0 | ✅ New |
| 12 | Bookworm | 3.11 | 6.15.9 | ✅ |
| 11 | Bullseye | 3.9 | 6.8.3 | ✅ |
| 10 | Buster | 3.7 | 6.8.3 | ⚠️ Legacy |

## Quick Install (From Releases)

### For Raspberry Pi Zero W (32-bit Trixie)

```bash
# Download packages from the latest release
RELEASE_URL="https://github.com/walkthru-earth/piaware_builder/releases/latest"

# Install dump1090-fa first
wget "${RELEASE_URL}/download/dump1090-fa-*-trixie-armhf.deb"
sudo dpkg -i dump1090-fa-*-trixie-armhf.deb

# Install piaware
wget "${RELEASE_URL}/download/piaware-*-trixie-armhf.deb"
sudo dpkg -i piaware-*-trixie-armhf.deb

# Fix any missing dependencies
sudo apt-get install -f

# Reboot
sudo reboot
```

### For Raspberry Pi 4/5 (64-bit Bookworm)

```bash
# Same process but use arm64 packages
wget "${RELEASE_URL}/download/dump1090-fa-*-bookworm-arm64.deb"
wget "${RELEASE_URL}/download/piaware-*-bookworm-arm64.deb"
sudo dpkg -i dump1090-fa-*.deb piaware-*.deb
sudo apt-get install -f
```

## Building Locally

### Prerequisites

```bash
sudo apt install build-essential git devscripts debhelper tcl8.6-dev autoconf \
  python3-dev python3-venv python3-setuptools libz-dev openssl \
  libboost-system-dev libboost-program-options-dev libboost-regex-dev \
  libboost-filesystem-dev patchelf
```

### Build for Your Distribution

```bash
# Clone this fork
git clone git@github.com:walkthru-earth/piaware_builder.git
cd piaware_builder

# Prepare the build (choose your distro)
./sensible-build.sh trixie    # For Debian 13
./sensible-build.sh bookworm  # For Debian 12
./sensible-build.sh bullseye  # For Debian 11

# Build the package
cd package-trixie  # or package-bookworm, etc.
dpkg-buildpackage -b --no-sign
```

### Cross-Compile with Docker (for armhf on x86)

```bash
# Install QEMU for ARM emulation
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

# Build for armhf
docker run --rm --platform=linux/arm/v7 \
  -v "$(pwd):/build" \
  debian:trixie \
  bash -c "cd /build && ./sensible-build.sh trixie && cd package-trixie && dpkg-buildpackage -b --no-sign"
```

## GitHub Actions Workflow

This fork includes an automated CI/CD workflow that:

1. **Triggers on:**
   - Push to `master`/`main`/`dev` branches
   - Tags matching `v*` (for releases)
   - Manual workflow dispatch (with distro/arch selection)

2. **Builds:**
   - PiAware package (includes fa-mlat-client, faup1090, faup978)
   - dump1090-fa package (standalone)

3. **Creates releases** when tags are pushed

### Manual Build Trigger

Go to **Actions** → **Build and Release PiAware Packages** → **Run workflow**

Select:
- **Debian version**: `trixie`, `bookworm`, `bullseye`, or `all`
- **Architecture**: `armhf`, `arm64`, `amd64`, or `all`

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Actions                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ generate-   │  │ build-      │  │ build-      │         │
│  │ matrix      │──│ piaware     │  │ dump1090    │         │
│  └─────────────┘  └──────┬──────┘  └──────┬──────┘         │
│                          │                │                 │
│                          ▼                ▼                 │
│                   ┌─────────────────────────┐               │
│                   │       release           │               │
│                   │  (on tag push)          │               │
│                   └─────────────────────────┘               │
└─────────────────────────────────────────────────────────────┘

Build Matrix (per job):
┌────────────┬────────────┬────────────┐
│  trixie    │  bookworm  │  bullseye  │
├────────────┼────────────┼────────────┤
│  armhf     │  armhf     │  armhf     │
│  arm64     │  arm64     │  arm64     │
│  amd64     │  amd64     │  amd64     │
└────────────┴────────────┴────────────┘
         = 9 build combinations
```

## Changelog (This Fork)

### v10.2-walkthru.1
- Added Debian 13 (Trixie) support
- Added `trixie/` configuration directory
- Updated `sensible-build.sh` with Trixie case
- Added cx_Freeze 7.2.0 for Python 3.12 compatibility
- Created GitHub Actions workflow for automated builds
- Added multi-architecture support (armhf, arm64, amd64)

## Related Repositories

- [walkthru-earth/dump1090](https://github.com/walkthru-earth/dump1090) - Our fork of dump1090-fa
- [walkthru-earth/piaware](https://github.com/walkthru-earth/piaware) - Our fork of piaware
- [flightaware/piaware_builder](https://github.com/flightaware/piaware_builder) - Upstream

## Contributing

1. Fork this repository
2. Create a feature branch
3. Make your changes
4. Test locally or wait for CI
5. Submit a pull request

## License

Same as upstream FlightAware projects. See original LICENSE files.

---

**Maintained by [walkthru-earth](https://github.com/walkthru-earth)** for the ADS-B research community.
