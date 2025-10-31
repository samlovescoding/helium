# Helium Browser - Documentation

Welcome to the Helium browser developer documentation! This documentation will help you understand, build, and contribute to Helium.

## 📚 Documentation Guide

### For New Developers

**Start here if you're new to Helium:**

1. **[Developer Guide](index.md)** - Understanding the project

   - Project overview and structure
   - Folder organization
   - Key concepts (patches, domain substitution, etc.)
   - Development workflow
   - Contributing guidelines

2. **[Installation Guide](install.md)** - Building Helium

   - System requirements
   - Step-by-step build instructions
   - Platform-specific guides (macOS, Linux, Windows)
   - Troubleshooting common issues

3. **[Patch Writing Tutorial](patch.md)** - Your first contribution
   - Hands-on tutorial for writing patches
   - Real-world examples
   - Testing and validation
   - Best practices

### Quick Links

| Document                                                          | Description                         | Audience            |
| ----------------------------------------------------------------- | ----------------------------------- | ------------------- |
| [index.md](index.md)                                              | Complete developer guide            | All developers      |
| [install.md](install.md)                                          | Build and installation instructions | New contributors    |
| [patch.md](patch.md)                                              | Tutorial for writing patches        | Feature developers  |
| [../.cursorrules](../.cursorrules) / [../CLAUDE.md](../CLAUDE.md) | AI assistant coding standards       | Cursor/Claude users |

## 🚀 Getting Started

### I Want To...

**...understand how Helium works**
→ Read the [Developer Guide](index.md)

**...build Helium from source**
→ Follow the [Installation Guide](install.md)

**...create a new feature or modify existing behavior**
→ Complete the [Patch Writing Tutorial](patch.md)

**...fix a bug**
→ Read [Patch Writing Tutorial](patch.md) for the process

**...update to a new Chromium version**
→ See [Developer Guide - Testing and Validation](index.md#testing-and-validation)

## 📖 Documentation Overview

### [index.md](index.md) - Developer Guide

**What's covered:**

- Project structure and organization
- Patch-based development methodology
- Key concepts (patches, domain substitution, resources)
- Development workflow
- Testing and validation
- Contributing guidelines

**Best for:**

- Understanding the codebase structure
- Learning Helium's architecture
- Planning new features
- Code review and maintenance

### [install.md](install.md) - Installation Guide

**What's covered:**

- Prerequisites and system requirements
- Downloading Chromium source
- Applying patches and domain substitution
- Building on different platforms
- Debug vs Release builds
- Troubleshooting build issues

**Best for:**

- First-time setup
- Building locally for testing
- Platform-specific build issues
- Optimizing build performance

### [patch.md](patch.md) - Patch Writing Tutorial

**What's covered:**

- What patches are and why we use them
- Finding code in Chromium
- Making changes to C++ source files
- Creating and formatting patch files
- Testing patches
- Common patterns and examples
- Best practices

**Best for:**

- Contributing features
- Modifying Chromium behavior
- Understanding existing patches
- Learning through hands-on examples

## 🔧 Development Resources

### Helium Repositories

- **[Helium (this repo)](https://github.com/imputnet/helium)** - Core patches and configuration
- [Helium for macOS](https://github.com/imputnet/helium-macos) - macOS builds and packaging
- [Helium for Linux](https://github.com/imputnet/helium-linux) - Linux builds and AppImage
- [Helium for Windows](https://github.com/imputnet/helium-windows) - Windows builds and installer
- [Helium Services](https://github.com/imputnet/helium-services) - Backend services
- [Helium Onboarding](https://github.com/imputnet/helium-onboarding) - First-run experience

### External Resources

- [Chromium Source](https://source.chromium.org/) - Browse Chromium code online
- [Chromium Development](https://www.chromium.org/developers/) - Official Chromium docs
- [ungoogled-chromium](https://github.com/ungoogled-software/ungoogled-chromium) - Our upstream project
- [ungoogled-chromium docs](https://ungoogled-software.github.io/ungoogled-chromium-wiki/) - Additional documentation

## 💡 Quick Reference

### Common Tasks

```bash
# Clone repository
git clone https://github.com/imputnet/helium.git
cd helium

# Download Chromium source
python3 utils/downloads.py -i downloads.ini -c chromium_version.txt

# Apply patches
python3 utils/patches.py apply /path/to/chromium/src patches/series

# Validate patches
python3 devutils/validate_patches.py

# Build Helium
cd /path/to/chromium/src
autoninja -C out/Default chrome
```

### Important Files

```
helium/
├── patches/series           # Order of patches to apply
├── chromium_version.txt     # Target Chromium version
├── downloads.ini            # Chromium download config
├── flags.gn                 # Build configuration
├── domain_regex.list        # Domain substitution patterns
├── domain_substitution.list # Files to process
└── pruning.list             # Files to remove from source
```

### Directory Structure

```
patches/
├── core/                    # Core privacy patches
├── extra/                   # Additional patches
├── helium/                  # Helium-specific patches
│   ├── core/               # Core functionality
│   ├── ui/                 # User interface
│   ├── settings/           # Settings page
│   └── hop/                # Onboarding
└── upstream-fixes/         # Build fixes
```

## 🤝 Contributing

Before submitting changes:

1. ✅ Read the [Developer Guide](index.md)
2. ✅ Follow the [Patch Writing Tutorial](patch.md)
3. ✅ Test your changes locally
4. ✅ Run validation scripts:
   ```bash
   python3 devutils/validate_patches.py
   python3 devutils/check_patch_files.py
   bash devutils/check_all_code.sh
   ```
5. ✅ Submit a pull request with clear description

## 📞 Getting Help

- **Questions?** Open a [GitHub Discussion](https://github.com/imputnet/helium/discussions)
- **Bug reports?** File an [Issue](https://github.com/imputnet/helium/issues)
- **Documentation issues?** Let us know via [Issues](https://github.com/imputnet/helium/issues)

## 📝 License

Helium-specific code and documentation is licensed under GPL-3.0.
See [../LICENSE](../LICENSE) for details.

---

**Happy developing!** 🚀 If you have suggestions for improving this documentation, please let us know!
