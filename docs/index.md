# Helium Browser - Developer Documentation

Welcome to the Helium browser development documentation. Helium is a privacy-focused Chromium-based web browser built on top of [ungoogled-chromium](https://github.com/ungoogled-software/ungoogled-chromium), designed with best privacy practices by default, unbiased ad-blocking, and a clean user experience.

## Table of Contents

- [Project Overview](#project-overview)
- [Project Structure](#project-structure)
- [Development Workflow](#development-workflow)
- [Key Concepts](#key-concepts)
- [Creating Features and Patches](#creating-features-and-patches)
- [Testing and Validation](#testing-and-validation)
- [Contributing Guidelines](#contributing-guidelines)

## Project Overview

Helium is based on Chromium version **142.0.7444.59** and inherits the ungoogled-chromium foundation which removes Google-specific dependencies and tracking features. This repository contains:

- Configuration files for building Chromium with Helium modifications
- Patch files that modify Chromium's behavior
- Resource files for branding (icons, logos)
- Utility scripts for development and building
- Domain substitution rules to block tracking

### Related Repositories

- **Main repo**: This repository (configuration, patches, resources)
- **Platform packaging**:
  - [helium-macos](https://github.com/imputnet/helium-macos)
  - [helium-linux](https://github.com/imputnet/helium-linux)
  - [helium-windows](https://github.com/imputnet/helium-windows)
- **Services**: [helium-services](https://github.com/imputnet/helium-services)
- **Onboarding**: [helium-onboarding](https://github.com/imputnet/helium-onboarding)
- **Extensions**: [ublock-origin-crx](https://github.com/imputnet/ublock-origin-crx)

## Project Structure

```
helium/
├── patches/                    # Patch files organized by category
│   ├── core/                  # Core patches (from ungoogled-chromium and others)
│   │   ├── ungoogled-chromium/  # Privacy and degoogling patches
│   │   ├── bromite/            # Patches from Bromite browser
│   │   ├── inox-patchset/      # Patches from Inox browser
│   │   └── iridium-browser/    # Patches from Iridium browser
│   ├── extra/                 # Additional optional patches
│   ├── helium/                # Helium-specific patches
│   │   ├── core/              # Core Helium functionality
│   │   ├── hop/               # Helium Onboarding Protocol
│   │   ├── settings/          # Settings customizations
│   │   └── ui/                # UI modifications
│   ├── upstream-fixes/        # Fixes for building issues
│   └── series                 # Ordered list of patches to apply
│
├── resources/                 # Branding and visual assets
│   ├── branding/             # App icons and logos
│   │   └── app_icon/         # Application icons
│   ├── favicons/             # Internal page favicons
│   ├── generate_resources.txt # Resource generation config
│   └── helium_resources.txt  # List of Helium-specific resources
│
├── utils/                    # Build and development utilities
│   ├── downloads.py          # Download Chromium source
│   ├── patches.py            # Apply patch files
│   ├── domain_substitution.py # Domain blocking/substitution
│   ├── prune_binaries.py     # Remove unnecessary files
│   ├── generate_resources.py # Generate branded resources
│   ├── replace_resources.py  # Replace Chromium resources
│   └── name_substitution.py  # Replace product names
│
├── devutils/                 # Development tools
│   ├── validate_patches.py   # Validate patch file integrity
│   ├── validate_config.py    # Validate configuration files
│   ├── update_lists.py       # Update configuration lists
│   └── check_*.py            # Various validation scripts
│
├── downloads.ini             # Chromium download configuration
├── flags.gn                  # GN build flags
├── domain_regex.list         # Domain substitution patterns
├── domain_substitution.list  # Files for domain substitution
├── pruning.list              # Files to remove from source tree
├── chromium_version.txt      # Target Chromium version
├── version.txt               # Helium major version
└── revision.txt              # Helium revision number
```

## Key Concepts

### 1. Patch-Based Development

Helium uses a patch-based approach to modify Chromium. Instead of forking the entire Chromium source, we maintain patch files that are applied to a clean Chromium source tree during the build process.

**Benefits:**

- Easy to update to new Chromium versions
- Clear documentation of all changes
- Smaller repository size
- Better collaboration and review process

### 2. Patch Categories

Patches are organized into categories in the `patches/` directory:

- **core/**: Essential patches inherited from ungoogled-chromium and other privacy-focused browsers
- **extra/**: Additional optional patches for enhanced functionality
- **helium/**: Helium-specific patches divided into:
  - `core/`: Core Helium features
  - `hop/`: Helium Onboarding Protocol
  - `settings/`: Settings page customizations
  - `ui/`: User interface modifications
- **upstream-fixes/**: Patches to fix build issues

### 3. Patch Series

The `patches/series` file defines the order in which patches are applied. This order is critical as some patches depend on others.

### 4. Domain Substitution

Domain substitution replaces Google domains and tracking URLs in the source code with blockable or neutral alternatives:

- `domain_regex.list`: Patterns to find and replace
- `domain_substitution.list`: Files to process

### 5. Resource Replacement

Helium replaces Chromium branding with its own:

- Application icons
- Product logos
- Internal page favicons (settings, history, bookmarks, etc.)

### 6. Binary Pruning

The `pruning.list` specifies test files, binaries, and unnecessary data to remove from the source tree, reducing build size and time.

## Development Workflow

### Setting Up Development Environment

See [install.md](install.md) for detailed installation instructions.

### Creating Features and Patches

#### 1. Modify Chromium Source

First, you need a working Chromium source tree with Helium patches applied:

```bash
# Download and prepare Chromium source (see install.md)
# Apply existing patches
python3 utils/patches.py apply /path/to/chromium/src
```

#### 2. Make Your Changes

Edit the Chromium source files directly to implement your feature:

```bash
cd /path/to/chromium/src
# Make your changes to the source files
```

#### 3. Generate a Patch File

After making changes, create a patch file:

```bash
# Create a patch from your changes
git diff > ~/my-feature.patch

# Or if you made commits:
git format-patch -1 HEAD --stdout > ~/my-feature.patch
```

#### 4. Add Patch to Helium

Move your patch to the appropriate location:

```bash
# For Helium-specific features, use helium/core/
cp ~/my-feature.patch patches/helium/core/feature-name.patch

# Or for UI changes:
cp ~/my-feature.patch patches/helium/ui/feature-name.patch
```

#### 5. Update Patch Series

Add your patch to `patches/series` in the appropriate location:

```
# ... existing patches ...
helium/core/feature-name.patch
# ... more patches ...
```

**Important**: Place your patch after any patches it depends on.

#### 6. Validate Your Patch

Use the validation tools to ensure your patch is correct:

```bash
# Validate patch file format
python3 devutils/validate_patches.py

# Check patch file paths exist
python3 devutils/check_patch_files.py

# Test applying patches to clean source
python3 utils/patches.py apply /path/to/clean/chromium/src
```

### Patch Naming Conventions

- Use lowercase with hyphens: `disable-feature-name.patch`
- Be descriptive but concise
- Group related patches with prefixes: `settings-disable-sync.patch`

### Patch File Format

Patches should be in unified diff format with these characteristics:

```diff
--- a/path/to/file.cc
+++ b/path/to/file.cc
@@ -10,7 +10,7 @@
 context line
 context line
-removed line
+added line
 context line
```

**Best practices:**

- Include sufficient context (3-5 lines before and after changes)
- Make patches as small and focused as possible
- One logical change per patch
- Add comments explaining non-obvious changes

## Testing and Validation

### Automated Validation

Run all validation checks before submitting changes:

```bash
# Validate all configuration files
python3 devutils/validate_config.py

# Check downloads.ini format
python3 devutils/check_downloads_ini.py

# Validate GN flags
python3 devutils/check_gn_flags.py

# Run all checks
bash devutils/check_all_code.sh
```

### Code Style

Helium uses standard Python tooling for utility scripts:

```bash
# Format Python code with YAPF
bash devutils/run_utils_yapf.sh

# Run pylint checks
bash devutils/run_utils_pylint.py

# Run tests
bash devutils/run_utils_tests.sh
```

### Testing Changes Locally

After creating a patch, test it by building Helium:

1. Apply patches to Chromium source
2. Build Chromium with Helium flags
3. Test the feature in the built browser
4. Verify no regressions in other features

See [install.md](install.md) for build instructions.

## Contributing Guidelines

### Before Creating a Pull Request

1. **Validate your changes**:

   ```bash
   bash devutils/check_all_code.sh
   ```

2. **Test your patch**:

   - Apply patches to clean Chromium source
   - Build successfully
   - Verify feature works as expected

3. **Document your changes**:

   - Add comments in patch files for complex changes
   - Update relevant documentation
   - Explain the purpose and benefit of the patch

4. **Follow conventions**:
   - Use appropriate patch directory
   - Follow naming conventions
   - Keep patches focused and minimal

### Feature Development Checklist

- [ ] Feature implemented and patch created
- [ ] Patch added to appropriate directory
- [ ] Patch added to `patches/series` file
- [ ] Validation scripts pass
- [ ] Builds successfully with patch applied
- [ ] Feature tested in built browser
- [ ] Documentation updated if needed
- [ ] Code style checks pass

## Additional Resources

- [Installation Guide](install.md) - How to build and test Helium locally
- [ungoogled-chromium documentation](https://github.com/ungoogled-software/ungoogled-chromium)
- [Chromium development documentation](https://www.chromium.org/developers/)

## Getting Help

- Check existing patches for examples
- Review ungoogled-chromium documentation
- Open an issue on GitHub for questions
- Join community discussions

## License

Helium-specific code is licensed under GPL-3.0. See [LICENSE](../LICENSE) for details.

Imported code retains its original license (e.g., ungoogled-chromium code remains BSD 3-Clause).
