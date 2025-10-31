# Helium Browser - Installation and Build Guide

This guide explains how to build Helium from source and test your changes locally. Building Helium involves downloading the Chromium source code, applying Helium patches, and compiling the browser.

## Table of Contents

- [Prerequisites](#prerequisites)
- [System Requirements](#system-requirements)
- [Quick Start](#quick-start)
- [Detailed Build Instructions](#detailed-build-instructions)
- [Platform-Specific Instructions](#platform-specific-instructions)
- [Development Build vs Release Build](#development-build-vs-release-build)
- [Testing Your Changes](#testing-your-changes)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### Common Requirements

- **Python 3.8+**: Required for Helium utility scripts
- **Git**: For version control and managing Chromium source
- **Sufficient disk space**: At least 100GB free (Chromium source is large)
- **RAM**: At least 16GB RAM (32GB recommended for faster builds)
- **Time**: Initial build can take 2-6 hours depending on your hardware

### Platform-Specific Requirements

<details>
<summary><b>macOS</b></summary>

- macOS 11 (Big Sur) or later
- Xcode Command Line Tools: `xcode-select --install`
- Homebrew (recommended): https://brew.sh

Install required tools:

```bash
# Using Homebrew
brew install python3 git ninja

# Optional but recommended
brew install ccache  # Speeds up rebuilds
```

</details>

<details>
<summary><b>Linux</b></summary>

- Ubuntu 20.04+ or equivalent distribution
- GCC 11+ or Clang 15+

Install required packages (Ubuntu/Debian):

```bash
sudo apt update
sudo apt install -y \
    python3 python3-pip git curl \
    build-essential ninja-build \
    libglib2.0-dev libnss3-dev \
    libatk1.0-dev libatk-bridge2.0-dev \
    libcups2-dev libdrm-dev libxkbcommon-dev \
    libgtk-3-dev

# Optional but recommended
sudo apt install ccache
```

For other distributions, refer to [Chromium Linux build requirements](https://chromium.googlesource.com/chromium/src/+/main/docs/linux/build_instructions.md).

</details>

<details>
<summary><b>Windows</b></summary>

- Windows 10 or later
- Visual Studio 2022 with C++ workload
- Windows 10 SDK

Install required tools:

1. Install [Visual Studio 2022 Community](https://visualstudio.microsoft.com/downloads/)
   - Include "Desktop development with C++" workload
   - Include Windows 10 SDK
2. Install [Python 3](https://www.python.org/downloads/)
3. Install [Git for Windows](https://git-scm.com/download/win)

**Note**: Windows builds are more complex. Consider using WSL2 with Linux instructions for easier setup.

</details>

## System Requirements

| Component  | Minimum                            | Recommended     |
| ---------- | ---------------------------------- | --------------- |
| CPU        | 4 cores                            | 8+ cores        |
| RAM        | 16 GB                              | 32 GB           |
| Disk Space | 100 GB                             | 150 GB+         |
| OS         | macOS 11, Ubuntu 20.04, Windows 10 | Latest versions |

## Quick Start

For experienced developers who want to get started quickly:

```bash
# 1. Clone Helium repository
git clone https://github.com/imputnet/helium.git
cd helium

# 2. Set up build directory
export CHROMIUM_SRC="$HOME/chromium/src"
mkdir -p "$(dirname "$CHROMIUM_SRC")"

# 3. Download Chromium source and dependencies
python3 utils/downloads.py -i downloads.ini -c chromium_version.txt \
    -o "$(dirname "$CHROMIUM_SRC")"

# 4. Download depot_tools (Chromium's build tools)
cd "$(dirname "$CHROMIUM_SRC")"
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="$PWD/depot_tools:$PATH"

# 5. Sync dependencies
cd "$CHROMIUM_SRC"
gclient sync --no-history

# 6. Apply Helium patches, domain substitution, and resources
cd /path/to/helium
python3 utils/domain_substitution.py apply -r domain_regex.list \
    -f domain_substitution.list -c /tmp/domsubcache "$CHROMIUM_SRC"
python3 utils/patches.py apply "$CHROMIUM_SRC" patches/series
python3 utils/generate_resources.py
python3 utils/replace_resources.py "$CHROMIUM_SRC"

# 7. Configure build
cd "$CHROMIUM_SRC"
gn gen out/Default --args="$(cat /path/to/helium/flags.gn)"

# 8. Build Helium
autoninja -C out/Default chrome

# 9. Run Helium
./out/Default/chrome  # or chrome.exe on Windows
```

## Detailed Build Instructions

### Step 1: Clone Helium Repository

```bash
git clone https://github.com/imputnet/helium.git
cd helium
```

This repository contains patches, configuration, and utilities needed to build Helium.

### Step 2: Set Up Directory Structure

Create a directory for the Chromium source:

```bash
# Choose a location with plenty of space
export CHROMIUM_SRC="$HOME/chromium/src"
mkdir -p "$(dirname "$CHROMIUM_SRC")"
```

**Note**: The Chromium source tree will be very large (50GB+).

### Step 3: Download Chromium Source

Helium uses the `utils/downloads.py` script to download the correct Chromium version:

```bash
python3 utils/downloads.py \
    -i downloads.ini \
    -c chromium_version.txt \
    -o "$(dirname "$CHROMIUM_SRC")"
```

**What this does:**

- Reads `chromium_version.txt` to get the target Chromium version
- Downloads the official Chromium source tarball
- Verifies checksums
- Extracts to the specified output directory

This will take some time depending on your internet connection (~2GB download).

### Step 4: Install depot_tools

Chromium uses Google's `depot_tools` for building:

```bash
cd "$(dirname "$CHROMIUM_SRC")"
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="$PWD/depot_tools:$PATH"
```

Add depot_tools to your PATH permanently:

```bash
# For bash
echo 'export PATH="$HOME/chromium/depot_tools:$PATH"' >> ~/.bashrc
source ~/.bashrc

# For zsh
echo 'export PATH="$HOME/chromium/depot_tools:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Step 5: Sync Chromium Dependencies

Chromium has many third-party dependencies. Use `gclient` to sync them:

```bash
cd "$CHROMIUM_SRC"
gclient sync --no-history
```

**Options explained:**

- `--no-history`: Shallow clone, saves time and space
- This step downloads additional dependencies (another ~10GB)

This will take 30-60 minutes depending on your connection.

### Step 6: Prune Unnecessary Files (Optional but Recommended)

Remove test files and binaries not needed for building:

```bash
cd /path/to/helium
python3 utils/prune_binaries.py "$CHROMIUM_SRC" pruning.list
```

This saves several GB of disk space and reduces build time.

### Step 7: Apply Domain Substitution

Replace Google domains and tracking URLs in the source:

```bash
python3 utils/domain_substitution.py apply \
    -r domain_regex.list \
    -f domain_substitution.list \
    -c /tmp/domsubcache \
    "$CHROMIUM_SRC"
```

**Parameters:**

- `-r`: Regex file with domain patterns
- `-f`: List of files to process
- `-c`: Cache directory (speeds up repeated runs)

### Step 8: Apply Patches

Apply all Helium and ungoogled-chromium patches:

```bash
python3 utils/patches.py apply "$CHROMIUM_SRC" patches/series
```

If a patch fails to apply, you'll see an error message. This usually happens when:

- Chromium version doesn't match
- Source files were modified
- Patches are out of order

### Step 9: Replace Resources and Branding

Generate Helium-branded resources:

```bash
# Generate resized icons
python3 utils/generate_resources.py

# Replace Chromium resources with Helium branding
python3 utils/replace_resources.py "$CHROMIUM_SRC"
```

This replaces Chromium icons, logos, and branding with Helium's.

### Step 10: Configure Build with GN

Configure the build using Helium's build flags:

```bash
cd "$CHROMIUM_SRC"
gn gen out/Default --args="import(\"//path/to/helium/flags.gn\")"
```

Or manually specify flags:

```bash
gn gen out/Default --args='
    is_debug=false
    is_official_build=true
    chrome_pgo_phase=0
    enable_widevine=true
    safe_browsing_mode=0
    use_unofficial_version_number=false
    google_api_key=""
    google_default_client_id=""
    google_default_client_secret=""
'
```

**Build types:**

- `out/Default`: Default configuration
- `out/Debug`: Debug build (slower, easier to debug)
- `out/Release`: Optimized release build

### Step 11: Build Helium

Build the browser using Ninja:

```bash
autoninja -C out/Default chrome
```

**Options:**

- `-C out/Default`: Specify build directory
- `chrome`: Target to build (the browser)
- `autoninja`: Wrapper that optimizes Ninja for your system

**Build time:**

- First build: 2-6 hours (depending on CPU)
- Incremental rebuilds: 5-30 minutes (only recompiles changed files)

**Tips for faster builds:**

- Use `ccache`: Caches compilation results
- Use more CPU cores: `autoninja` does this automatically
- Use a faster SSD
- Close other applications to free up RAM

### Step 12: Run Helium

After building, run your locally built Helium:

```bash
cd "$CHROMIUM_SRC"
./out/Default/chrome
```

On macOS:

```bash
./out/Default/Chromium.app/Contents/MacOS/Chromium
```

On Windows:

```cmd
out\Default\chrome.exe
```

## Platform-Specific Instructions

### macOS

**Additional setup:**

1. After first build, create an Xcode project (optional, for IDE):

   ```bash
   gn gen out/Default --ide=xcode
   open out/Default/all.xcodeproj
   ```

2. Code signing (for distribution):

   ```bash
   # Set up signing identity in GN args
   gn gen out/Default --args='
       is_official_build=true
       mac_signing_identity="Developer ID Application: Your Name"
   '
   ```

3. Create app bundle:
   ```bash
   autoninja -C out/Default chrome
   # App is at out/Default/Helium.app
   ```

### Linux

**Creating AppImage:**

For distribution, you may want to create an AppImage. Refer to the [helium-linux](https://github.com/imputnet/helium-linux) repository for packaging scripts.

**Desktop entry:**

```bash
# Install locally (optional)
sudo cp out/Default/chrome /opt/helium/helium
sudo cp resources/branding/product_logo.png /opt/helium/
```

### Windows

**Visual Studio integration:**

```cmd
gn gen out\Default --ide=vs
start out\Default\all.sln
```

**Running with debugging:**

```cmd
out\Default\chrome.exe --enable-logging --v=1
```

## Development Build vs Release Build

### Debug Build

Good for development and testing:

```bash
gn gen out/Debug --args='
    is_debug=true
    symbol_level=2
    enable_nacl=false
'
```

**Characteristics:**

- Includes debug symbols
- No optimizations
- Easier to debug with GDB/LLDB
- Much larger binary size
- Slower runtime performance

### Release Build

For testing production performance:

```bash
gn gen out/Release --args='
    is_debug=false
    is_official_build=true
    symbol_level=0
'
```

**Characteristics:**

- Fully optimized
- Smaller binary size
- Faster performance
- Harder to debug

## Testing Your Changes

### Testing a Patch

After creating a new patch:

1. **Clean build directory:**

   ```bash
   rm -rf out/Default
   ```

2. **Revert Chromium source:**

   ```bash
   cd "$CHROMIUM_SRC"
   git reset --hard HEAD
   git clean -fd
   ```

3. **Reapply patches with your changes:**

   ```bash
   cd /path/to/helium
   python3 utils/patches.py apply "$CHROMIUM_SRC" patches/series
   ```

4. **Rebuild:**

   ```bash
   cd "$CHROMIUM_SRC"
   gn gen out/Default --args="import(\"//path/to/helium/flags.gn\")"
   autoninja -C out/Default chrome
   ```

5. **Test your feature:**
   ```bash
   ./out/Default/chrome
   ```

### Running Tests

Chromium has extensive test suites:

```bash
# Run unit tests
autoninja -C out/Default unit_tests
./out/Default/unit_tests

# Run browser tests
autoninja -C out/Default browser_tests
./out/Default/browser_tests --gtest_filter="YourTest*"
```

## Troubleshooting

### Build Fails with "patch does not apply"

**Solution:**

- Ensure Chromium version matches `chromium_version.txt`
- Check if patches are in correct order in `patches/series`
- Validate patches: `python3 devutils/validate_patches.py`

### Out of Memory During Build

**Solutions:**

- Close other applications
- Use fewer parallel jobs: `ninja -C out/Default -j4 chrome`
- Add swap space (Linux)
- Use a debug build which requires less RAM

### "depot_tools not found"

**Solution:**

```bash
export PATH="$HOME/chromium/depot_tools:$PATH"
```

Add to your shell rc file to make permanent.

### GN Args Error

**Solution:**
Check your GN args syntax:

```bash
gn args out/Default --list
gn args out/Default
```

### Slow Build Times

**Solutions:**

- Use ccache: `export CCACHE_DIR=/path/to/cache`
- Use component build: `is_component_build=true` (faster linking)
- Only build what you need: `autoninja -C out/Default chrome` (not `all`)

### Chromium Source Download Fails

**Solution:**

- Check internet connection
- Try download again (script supports resume)
- Manually download from: https://commondatastorage.googleapis.com/chromium-browser-official/

### Cannot Run Built Browser

**macOS:**

```bash
# Remove quarantine attribute
xattr -cr out/Default/Chromium.app
```

**Linux:**

```bash
# Check library dependencies
ldd out/Default/chrome
```

## Next Steps

After successfully building Helium:

1. **Read the development guide**: See [index.md](index.md) for how to create patches
2. **Make your first change**: Try modifying a patch and rebuilding
3. **Join the community**: Contribute to Helium's development

## Additional Resources

- [Chromium build documentation](https://chromium.googlesource.com/chromium/src/+/main/docs/README.md)
- [ungoogled-chromium build guide](https://github.com/ungoogled-software/ungoogled-chromium/blob/master/docs/building.md)
- [GN reference](https://gn.googlesource.com/gn/+/main/docs/reference.md)
- [Ninja manual](https://ninja-build.org/manual.html)

## Getting Help

If you encounter issues not covered here:

- Check [GitHub Issues](https://github.com/imputnet/helium/issues)
- Review Chromium and ungoogled-chromium documentation
- Ask in the community discussions

Happy building! 🚀
