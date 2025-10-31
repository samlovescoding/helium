# Writing Your First Helium Patch - A Tutorial

Welcome! This tutorial will guide you through writing a patch for Helium browser. We'll walk through a real example from start to finish, helping you understand the process of modifying Chromium's behavior through patches.

## Table of Contents

- [What Are Patches?](#what-are-patches)
- [Prerequisites](#prerequisites)
- [Tutorial Overview](#tutorial-overview)
- [Part 1: Setting Up Your Environment](#part-1-setting-up-your-environment)
- [Part 2: Understanding the Chromium Codebase](#part-2-understanding-the-chromium-codebase)
- [Part 3: Making Your Changes](#part-3-making-your-changes)
- [Part 4: Creating the Patch File](#part-4-creating-the-patch-file)
- [Part 5: Integrating Your Patch](#part-5-integrating-your-patch)
- [Part 6: Testing and Validation](#part-6-testing-and-validation)
- [Common Patterns and Examples](#common-patterns-and-examples)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## What Are Patches?

If you're coming from web development, think of patches as "surgical modifications" to Chromium's source code. Instead of forking the entire Chromium repository, we maintain small patch files that describe specific changes.

**Why patches?**

- **Maintainability**: Easy to update when new Chromium versions are released
- **Transparency**: Each patch documents a specific change with clear intent
- **Modularity**: Patches can be added, removed, or reordered independently
- **Review**: Easier to review and understand changes

A patch file is essentially a "diff" - it shows:

- Which files to modify
- What lines to remove (marked with `-`)
- What lines to add (marked with `+`)
- Context lines around changes (unmarked)

## Prerequisites

Before starting this tutorial, ensure you have:

1. ✅ Built Helium successfully (see [install.md](install.md))
2. ✅ Basic understanding of C++ (Chromium is written in C++)
3. ✅ Familiarity with Git and diff tools
4. ✅ Text editor or IDE of your choice
5. ✅ Chromium source directory with Helium patches applied

**Environment variables we'll use:**

```bash
export HELIUM_ROOT="/path/to/helium"
export CHROMIUM_SRC="/path/to/chromium/src"
```

## Tutorial Overview

**What we'll build:** A patch that disables the "Restore pages?" dialog when Chromium crashes.

**Why this example?**

- Simple enough to understand
- Touches UI components
- Demonstrates common patch patterns
- Useful feature for a privacy browser

**Skills you'll learn:**

- Finding relevant code in Chromium
- Modifying C++ source files
- Creating properly formatted patches
- Testing your changes
- Integrating patches into Helium

Let's get started! 🚀

## Part 1: Setting Up Your Environment

### Step 1: Create a Clean Working Branch

Navigate to your Chromium source directory and create a branch for your work:

```bash
cd "$CHROMIUM_SRC"
git checkout -b feature/disable-restore-dialog
```

**Why a branch?** This keeps your changes isolated and makes it easy to generate patches later.

### Step 2: Verify Current State

Make sure all existing Helium patches are applied:

```bash
cd "$HELIUM_ROOT"
python3 utils/patches.py apply "$CHROMIUM_SRC" patches/series
```

If patches fail to apply, start fresh:

```bash
cd "$CHROMIUM_SRC"
git reset --hard HEAD
git clean -fd
cd "$HELIUM_ROOT"
python3 utils/patches.py apply "$CHROMIUM_SRC" patches/series
```

### Step 3: Set Up Your Editor

For C++ development in Chromium, consider using:

- **VS Code** with C++ extensions
- **CLion** (JetBrains IDE)
- **Vim/Neovim** with LSP
- **Xcode** (macOS)

Make sure you can navigate the codebase comfortably.

## Part 2: Understanding the Chromium Codebase

### Chromium Directory Structure (Quick Reference)

```
chromium/src/
├── chrome/                 # Chrome-specific code (browser UI, features)
│   ├── browser/           # Browser process code (main UI)
│   ├── renderer/          # Renderer process code (web content)
│   ├── common/            # Code shared between processes
│   └── app/               # Application initialization
├── content/               # Multi-process architecture core
├── components/            # Reusable components
├── ui/                    # UI framework and widgets
├── net/                   # Network stack
├── base/                  # Fundamental utilities
└── third_party/           # External dependencies
```

### Finding Code: Detective Work

Our goal: Disable the crash restore dialog. Let's find where it's implemented.

**Strategy 1: Search for UI strings**

The dialog shows "Restore pages?" - let's search for that:

```bash
cd "$CHROMIUM_SRC"
git grep -i "restore pages" -- "*.cc" "*.h" "*.grd"
```

**Strategy 2: Search for related functions**

Crash recovery likely involves session restoration:

```bash
git grep -i "session.*crash" -- "chrome/browser/*.cc"
git grep -i "restore.*after.*crash" -- "*.cc"
```

**Strategy 3: Use code search tools**

Chromium Code Search: https://source.chromium.org/chromium

Search for: `"restore pages" crash`

### What We Found

After searching, we discover the relevant code is in:

- `chrome/browser/sessions/session_restore.cc` - Main session restore logic
- `chrome/browser/ui/startup/startup_browser_creator_impl.cc` - Startup handling
- `chrome/browser/ui/views/session_crashed_bubble_view.cc` - The dialog itself!

Let's examine `session_crashed_bubble_view.cc`:

```bash
cd "$CHROMIUM_SRC"
less chrome/browser/ui/views/session_crashed_bubble_view.cc
```

You'll see something like:

```cpp
// Function that shows the crashed bubble
void SessionCrashedBubbleView::Show(Browser* browser) {
  // ... code that displays the dialog
}
```

Perfect! This is what we need to modify.

## Part 3: Making Your Changes

### Understanding the Code

Open the file in your editor:

```bash
code chrome/browser/ui/views/session_crashed_bubble_view.cc
# or vim, emacs, etc.
```

Look for the `Show` function or similar. You'll see code that creates and displays the dialog.

### Option 1: Disable the Dialog Completely

The simplest approach - make the `Show` function return early:

```cpp
void SessionCrashedBubbleView::Show(Browser* browser) {
  // Helium: Disable crash restore dialog for privacy
  return;

  // Original code below (now unreachable)
  // ... existing implementation ...
}
```

### Option 2: Change Default Behavior

Or, automatically restore without asking:

```cpp
void SessionCrashedBubbleView::Show(Browser* browser) {
  // Helium: Auto-restore without showing dialog
  SessionRestore::RestoreSessionAfterCrash(browser);
  return;

  // Original code below
  // ... existing implementation ...
}
```

### Making the Edit

For this tutorial, let's disable it completely. Edit the file:

```cpp
// Find the Show function (around line 150-200, varies by version)
void SessionCrashedBubbleView::Show(Browser* browser) {
  // Helium: Disable crash restore dialog
  // Users can still restore via Menu > History > Restore tabs
  return;

  // [Original code remains below but won't execute]
```

**Important:**

- Add clear comments explaining WHY you made the change
- Mention it's a Helium modification
- Suggest alternative ways to achieve the same functionality

### Save Your Changes

Save the file and verify your edit:

```bash
git diff chrome/browser/ui/views/session_crashed_bubble_view.cc
```

You should see your changes marked with `+` and `-`.

## Part 4: Creating the Patch File

### Commit Your Changes

First, commit your changes to Git:

```bash
cd "$CHROMIUM_SRC"
git add chrome/browser/ui/views/session_crashed_bubble_view.cc
git commit -m "Disable crash restore dialog

Users can still restore tabs via Menu > History > Restore tabs.
This reduces interruption and maintains privacy-first approach."
```

**Commit message tips:**

- First line: Brief summary (50 chars or less)
- Blank line
- Detailed explanation of what and why

### Generate the Patch File

Now create a patch file from your commit:

```bash
git format-patch -1 HEAD --stdout > ~/disable-crash-restore-dialog.patch
```

Let's break down this command:

- `format-patch`: Git command to create patch files
- `-1 HEAD`: Create patch from the last 1 commit
- `--stdout`: Output to terminal instead of file
- `> ~/disable-crash-restore-dialog.patch`: Redirect to file

### Inspect Your Patch

Look at the patch file you created:

```bash
cat ~/disable-crash-restore-dialog.patch
```

You should see something like:

```diff
From 1234567890abcdef1234567890abcdef12345678 Mon Sep 17 00:00:00 2001
From: Your Name <your.email@example.com>
Date: Fri, 31 Oct 2025 10:00:00 +0000
Subject: [PATCH] Disable crash restore dialog

Users can still restore tabs via Menu > History > Restore tabs.
This reduces interruption and maintains privacy-first approach.
---
 chrome/browser/ui/views/session_crashed_bubble_view.cc | 4 ++++
 1 file changed, 4 insertions(+)

diff --git a/chrome/browser/ui/views/session_crashed_bubble_view.cc b/chrome/browser/ui/views/session_crashed_bubble_view.cc
index 1234567..abcdefg 100644
--- a/chrome/browser/ui/views/session_crashed_bubble_view.cc
+++ b/chrome/browser/ui/views/session_crashed_bubble_view.cc
@@ -150,6 +150,10 @@ SessionCrashedBubbleView::~SessionCrashedBubbleView() {
 }

 void SessionCrashedBubbleView::Show(Browser* browser) {
+  // Helium: Disable crash restore dialog
+  // Users can still restore via Menu > History > Restore tabs
+  return;
+
   if (!browser->is_type_normal())
     return;
```

**Key parts of a patch:**

- **Header**: Metadata (author, date, commit message)
- **File path**: `diff --git a/... b/...`
- **Hunk header**: `@@` lines showing line numbers
- **Changes**: Lines with `-` (removed) and `+` (added)

### Clean Up the Patch (Optional)

Git format-patch includes commit metadata. For Helium patches, you can simplify:

```bash
# Remove the email header and keep only the diff
cd "$CHROMIUM_SRC"
git diff HEAD~1 HEAD > ~/disable-crash-restore-dialog.patch
```

This creates a cleaner patch with just the diff.

## Part 5: Integrating Your Patch

### Step 1: Move Patch to Helium Repository

Copy your patch to the appropriate location:

```bash
cp ~/disable-crash-restore-dialog.patch \
   "$HELIUM_ROOT/patches/helium/ui/disable-crash-restore-dialog.patch"
```

**Choosing the right directory:**

- `helium/core/`: Core functionality changes
- `helium/ui/`: User interface modifications ← **We use this**
- `helium/settings/`: Settings page changes
- `helium/hop/`: Onboarding-related changes

### Step 2: Add to Patch Series

Edit `patches/series` to include your new patch:

```bash
cd "$HELIUM_ROOT"
nano patches/series
# or: code patches/series
```

Find the section with `helium/ui/` patches and add yours:

```
# ... other patches ...

# Helium UI patches
helium/ui/disable-crash-restore-dialog.patch
helium/ui/customize-new-tab-page.patch
# ... more patches ...
```

**Order matters!** Place your patch:

- After patches it depends on
- Before patches that depend on it
- Grouped with related patches

### Step 3: Validate Your Patch

Use Helium's validation tools:

```bash
cd "$HELIUM_ROOT"

# Validate patch file format
python3 devutils/validate_patches.py

# Check that all patch files exist
python3 devutils/check_patch_files.py
```

If validation passes, you're good! If not, fix the reported issues.

### Step 4: Test Applying Patches

Test on a clean Chromium source:

```bash
# Reset to clean state
cd "$CHROMIUM_SRC"
git reset --hard HEAD
git clean -fd

# Apply all patches including yours
cd "$HELIUM_ROOT"
python3 utils/patches.py apply "$CHROMIUM_SRC" patches/series
```

Look for success messages:

```
Applying patch: helium/ui/disable-crash-restore-dialog.patch ... OK
```

If it fails:

```
Applying patch: helium/ui/disable-crash-restore-dialog.patch ... FAILED
```

Fix the patch and try again.

## Part 6: Testing and Validation

### Step 1: Rebuild Chromium

With patches applied, rebuild:

```bash
cd "$CHROMIUM_SRC"

# Clean build (ensures no stale artifacts)
rm -rf out/Default
gn gen out/Default --args='is_debug=true'

# Build (this will take time)
autoninja -C out/Default chrome
```

**Debug build tips:**

- Use `is_debug=true` for development
- Faster to compile than release
- Better error messages

### Step 2: Manual Testing

Run your built browser:

```bash
./out/Default/chrome
```

**Test your patch:**

1. **Simulate a crash**:

   - Open several tabs
   - Open Chrome Task Manager: `Shift + Esc`
   - Kill the browser process

2. **Restart the browser**:

   - Launch again
   - Observe: The "Restore pages?" dialog should NOT appear ✅

3. **Verify alternative works**:
   - Menu > History > Should see recently closed tabs
   - Click to restore manually

### Step 3: Test Edge Cases

Try different scenarios:

**Test Case 1: Clean start**

```bash
./out/Default/chrome --user-data-dir=/tmp/test-profile
```

No dialog should appear on first run ✅

**Test Case 2: Graceful shutdown**

```bash
# Open browser, close normally
# Reopen - should start normally
```

No issues ✅

**Test Case 3: Multiple crashes**

```bash
# Crash multiple times
# Each restart should not show dialog
```

Works correctly ✅

### Step 4: Check for Side Effects

Make sure your patch doesn't break other features:

- [ ] Settings still load correctly
- [ ] History menu accessible
- [ ] No console errors on startup
- [ ] Extensions load properly
- [ ] Profile switching works

### Step 5: Run Automated Tests (Optional)

If you modified critical code, run Chromium's tests:

```bash
# Build tests
autoninja -C out/Default unit_tests

# Run relevant tests
./out/Default/unit_tests \
  --gtest_filter="*SessionCrashed*"
```

## Common Patterns and Examples

### Pattern 1: Disabling a Feature

**Problem:** Remove an unwanted feature

**Solution:** Make the feature's entry point return early

```cpp
void UnwantedFeature::Initialize() {
  // Helium: Disable unwanted feature
  return;

  // Original code...
}
```

**Example patches in Helium:**

- `disable-crash-reporter.patch`
- `disable-google-host-detection.patch`

### Pattern 2: Changing Default Values

**Problem:** Change a default setting

**Solution:** Modify the default constant

```cpp
// Before
const bool kDefaultEnableFeature = true;

// After
const bool kDefaultEnableFeature = false;  // Helium: Disable by default
```

**Example patches:**

- `modify-default-prefs.patch`

### Pattern 3: Removing UI Elements

**Problem:** Hide a UI component

**Solution:** Return early from the UI creation function

```cpp
std::unique_ptr<views::View> CreateUnwantedButton() {
  // Helium: Remove this button
  return nullptr;

  // Original button creation...
}
```

### Pattern 4: Blocking Network Requests

**Problem:** Prevent specific network calls

**Solution:** Add early return in network request handler

```cpp
void MakeTrackingRequest(const GURL& url) {
  // Helium: Block tracking requests
  if (url.host().find("tracking-domain.com") != std::string::npos) {
    return;
  }

  // Original request code...
}
```

**Example patches:**

- `block-trk-and-subdomains.patch`

### Pattern 5: Adding Command-Line Flags

**Problem:** Make feature togglable via flag

**Solution:** Check for flag before executing

```cpp
void ConditionalFeature::Run() {
  // Helium: Allow disabling via flag
  if (base::CommandLine::ForCurrentProcess()->
        HasSwitch("disable-conditional-feature")) {
    return;
  }

  // Original feature code...
}
```

**Example patches:**

- `toggle-translation-via-switch.patch`

## Real-World Example: Customizing the New Tab Page

Let's walk through another example - customizing the New Tab Page (NTP).

### Goal

Remove promotional content from the New Tab Page.

### Step 1: Find the Code

```bash
cd "$CHROMIUM_SRC"
git grep -i "new tab page" chrome/browser/ui/webui/
```

Found: `chrome/browser/ui/webui/new_tab_page/new_tab_page_handler.cc`

### Step 2: Examine the File

```cpp
// In new_tab_page_handler.cc
void NewTabPageHandler::GetPromoContent(
    GetPromoContentCallback callback) {
  // Fetches promotional content
  FetchPromoFromServer(std::move(callback));
}
```

### Step 3: Modify

```cpp
void NewTabPageHandler::GetPromoContent(
    GetPromoContentCallback callback) {
  // Helium: Disable promo content on NTP
  std::move(callback).Run(nullptr);
  return;

  // Original code
  // FetchPromoFromServer(std::move(callback));
}
```

### Step 4: Create Patch

```bash
git add chrome/browser/ui/webui/new_tab_page/new_tab_page_handler.cc
git commit -m "Remove promotional content from New Tab Page"
git diff HEAD~1 HEAD > "$HELIUM_ROOT/patches/helium/ui/ntp-remove-promo.patch"
```

### Step 5: Add to Series

```bash
echo "helium/ui/ntp-remove-promo.patch" >> "$HELIUM_ROOT/patches/series"
```

### Step 6: Test

```bash
cd "$CHROMIUM_SRC"
git reset --hard && git clean -fd
cd "$HELIUM_ROOT"
python3 utils/patches.py apply "$CHROMIUM_SRC" patches/series
cd "$CHROMIUM_SRC"
autoninja -C out/Default chrome
./out/Default/chrome
```

Open a new tab - no promos! ✅

## Troubleshooting

### Patch Doesn't Apply

**Error:**

```
error: patch failed: chrome/browser/file.cc:123
error: chrome/browser/file.cc: patch does not apply
```

**Solutions:**

1. **Check Chromium version:**

   ```bash
   cd "$CHROMIUM_SRC"
   git log --oneline -1
   ```

   Make sure it matches `chromium_version.txt`

2. **Check line numbers:**
   The file may have changed. Manually apply the patch:

   - Open the file
   - Find the relevant section
   - Make changes manually
   - Regenerate patch

3. **Check for conflicts:**
   Another patch may have modified the same area. Reorder patches or merge changes.

### Build Fails After Applying Patch

**Error:**

```
error: 'ClassName' was not declared in this scope
```

**Solutions:**

1. **Missing includes:**
   Add required header files:

   ```cpp
   #include "base/feature_list.h"
   #include "components/prefs/pref_service.h"
   ```

2. **Check syntax:**
   Make sure your C++ syntax is correct:

   ```bash
   # Check with clang-format
   clang-format -i path/to/modified/file.cc
   ```

3. **Rebuild from scratch:**
   ```bash
   rm -rf out/Default
   gn gen out/Default
   autoninja -C out/Default chrome
   ```

### Feature Still Appears

**Problem:** Made changes but feature still works

**Possible causes:**

1. **Wrong file modified:**

   - Double-check you're editing the right file
   - Search for other places the feature might be implemented

2. **Code path not executed:**

   - Add logging to verify your code runs:

   ```cpp
   LOG(INFO) << "Helium: Feature disabled";
   ```

   - Run with logging: `./chrome --enable-logging --v=1`

3. **Cached artifacts:**
   - Clean build and rebuild:
   ```bash
   rm -rf out/Default
   autoninja -C out/Default chrome
   ```

### Patch Conflicts with Other Patches

**Error:**

```
Hunk #1 FAILED at 123.
```

**Solutions:**

1. **Reorder patches:**
   Move your patch earlier or later in `patches/series`

2. **Merge changes:**
   If both patches modify the same code:

   - Apply first patch manually
   - Make your changes on top
   - Create new combined patch

3. **Split your patch:**
   If it touches multiple unrelated areas:
   - Create separate patches for each change
   - Easier to maintain and debug

## Best Practices

### 1. Keep Patches Small and Focused

✅ **Good:**

```
patches/helium/ui/disable-crash-dialog.patch  (changes 1 file, 5 lines)
```

❌ **Bad:**

```
patches/helium/ui/various-ui-changes.patch  (changes 20 files, 500 lines)
```

### 2. Use Descriptive Names

✅ **Good:**

- `disable-google-analytics.patch`
- `remove-promo-content-ntp.patch`
- `change-default-search-engine.patch`

❌ **Bad:**

- `fix.patch`
- `change.patch`
- `ui-patch-2.patch`

### 3. Add Clear Comments

```cpp
// Helium: Disable feature X because it compromises privacy
// Users can still access this via Menu > Tools > Feature X
return;
```

**Include:**

- **Why** the change is made
- **What** alternative exists (if any)
- **Reference** to related issue/discussion (if applicable)

### 4. Test Thoroughly

Create a testing checklist:

```markdown
## Testing Checklist for patch-name.patch

- [ ] Clean build succeeds
- [ ] Feature behaves as expected
- [ ] No console errors
- [ ] Doesn't break related features
- [ ] Works across all platforms (if applicable)
- [ ] Gracefully handles edge cases
```

### 5. Document Complex Changes

For non-obvious changes, add documentation:

```cpp
// Helium: Custom implementation of session restore
//
// Instead of Chromium's default behavior (show dialog), we:
// 1. Check if restore is enabled in preferences
// 2. If yes, silently restore tabs
// 3. If no, start with empty NTP
//
// This maintains privacy while allowing power users to enable
// restore via Settings > Privacy > Restore tabs after crash
```

### 6. Follow Chromium Coding Style

Helium patches should follow Chromium's style (documented in `.cursorrules`):

- **Indentation:** 2 spaces (not tabs)
- **Line length:** 80 characters
- **Naming:** `PascalCase` for classes/functions, `snake_case_` for members, `kConstant` for constants
- **Comments:** `//` for single-line, `/* */` for multi-line
- **Helium prefix:** Always prefix Helium comments with `// Helium:`

Check style:

```bash
git clang-format --style=Chromium
```

**Tip:** The project's `.cursorrules` file contains comprehensive style guidelines for all file types.

### 7. Version Control Your Patches

Keep track of patch changes:

```bash
cd "$HELIUM_ROOT"
git add patches/helium/ui/disable-crash-restore-dialog.patch
git commit -m "Add patch to disable crash restore dialog"
```

### 8. Test on Clean Source

Always test patches on a clean Chromium source:

```bash
# Before submitting, verify:
cd "$CHROMIUM_SRC"
git reset --hard HEAD && git clean -fd
cd "$HELIUM_ROOT"
python3 utils/patches.py apply "$CHROMIUM_SRC" patches/series
# Should succeed without errors
```

## Next Steps

Congratulations! You've learned how to create patches for Helium. 🎉

**To continue your journey:**

1. **Study existing patches:**

   ```bash
   ls -la "$HELIUM_ROOT/patches/helium/"
   ```

   Learn from real examples in the codebase

2. **Start small:**

   - Modify default preferences
   - Change UI strings
   - Remove simple features

3. **Graduate to complex patches:**

   - Add new features
   - Modify network behavior
   - Create new UI components

4. **Contribute back:**
   - Submit pull requests
   - Share your patches with the community
   - Help others learn

## Additional Resources

- [Chromium Codebase](https://source.chromium.org/) - Browse Chromium source online
- [Chromium Development](https://www.chromium.org/developers/) - Official dev docs
- [Git Format-Patch](https://git-scm.com/docs/git-format-patch) - Git patch documentation
- [Helium GitHub](https://github.com/imputnet/helium) - Submit issues and PRs

## Getting Help

- **Check existing patches:** Many problems are already solved
- **Search Chromium docs:** Comprehensive documentation available
- **Ask the community:** GitHub issues and discussions
- **Debug with logs:** Use `LOG(INFO)` liberally while developing

Happy patching! 🚀
