# asdf-herdr Modernization Plan

This document outlines the changes needed to update `asdf-herdr` to follow modern asdf plugin practices, particularly after the asdf 0.17+ rewrite as a standalone binary.

## Current State Analysis

The current plugin follows the **legacy asdf plugin structure** (pre-0.17):
- `bin/download` - Downloads the binary
- `bin/install` - Installs the binary to the install path
- `bin/list-all` - Lists available versions
- `bin/latest-stable` - Resolves the latest stable version
- `lib/utils.bash` - Shared utility functions

This is still **functional** but misses several modern features and best practices.

## Changes Required

### 1. Add `plugin.toml` (REQUIRED for asdf 0.17+)

**What:** Create a `plugin.toml` file at the repository root.

**Why:** asdf 0.17+ requires this file for plugin metadata and validation.

**Format:**
```toml
name = "herdr"
description = "asdf plugin for herdr"
repository = "https://github.com/ogulcancelik/herdr"
extensions = []
aliases = {}
```

**Actions:**
- [ ] Create `plugin.toml` with required metadata
- [ ] Add any optional fields (e.g., `extensions` if needed)

---

### 2. Update `bin/download` (MODERNIZE)

**What:** Keep the core logic but ensure it aligns with asdf 0.17+ expectations.

**Why:** The asdf binary now handles more boilerplate, but the download script still needs to place the binary in `$ASDF_DOWNLOAD_PATH/bin/$TOOL_NAME`.

**Current issues:**
- The script is minimal and correct, but could benefit from better error handling.

**Actions:**
- [ ] Verify `bin/download` places binary in `$ASDF_DOWNLOAD_PATH/bin/herdr`
- [ ] Add explicit error handling for download failures
- [ ] Consider adding a checksum verification step (if herdr provides them)

**Modern pattern:**
```bash
#!/usr/bin/env bash
set -euo pipefail

current_script_path=${BASH_SOURCE[0]}
plugin_dir=$(dirname "$(dirname "$current_script_path")")
# shellcheck source=../lib/utils.bash
source "${plugin_dir}/lib/utils.bash"

mkdir -p "$ASDF_DOWNLOAD_PATH/bin"
binary_path="$ASDF_DOWNLOAD_PATH/bin/$TOOL_NAME"

download_release "$ASDF_INSTALL_VERSION" "$binary_path"
chmod +x "$binary_path"

# Optional: verify checksum if available
# if [ -n "${CHECKSUM:-}" ]; then
#   echo "$CHECKSUM  $binary_path" | sha256sum -c
# fi
```

---

### 3. Update `bin/install` (MODERNIZE)

**What:** Keep the core logic but ensure it aligns with asdf 0.17+ expectations.

**Why:** asdf 0.17+ expects the install script to copy the binary to `$ASDF_INSTALL_PATH/bin/` and make it executable.

**Current issues:**
- The script is minimal and correct.

**Actions:**
- [ ] Verify `bin/install` copies binary to `${ASDF_INSTALL_PATH%/bin}/bin/herdr`
- [ ] Add validation that the binary is executable after install
- [ ] Consider adding a `bin/test` hook (see below)

**Modern pattern:**
```bash
#!/usr/bin/env bash
set -euo pipefail

current_script_path=${BASH_SOURCE[0]}
plugin_dir=$(dirname "$(dirname "$current_script_path")")
# shellcheck source=../lib/utils.bash
source "${plugin_dir}/lib/utils.bash"

install_version "$ASDF_INSTALL_TYPE" "$ASDF_INSTALL_VERSION" "$ASDF_INSTALL_PATH"
```

---

### 4. Update `bin/list-all` (MODERNIZE)

**What:** Keep the core logic but ensure it aligns with asdf 0.17+ expectations.

**Why:** asdf 0.17+ expects `list-all` to output one version per line, and asdf handles sorting.

**Current issues:**
- The script uses `sort_versions | xargs echo` which is redundant (asdf sorts now).

**Actions:**
- [ ] Remove `sort_versions` and `xargs echo` (asdf handles sorting)
- [ ] Output one version per line directly

**Modern pattern:**
```bash
#!/usr/bin/env bash
set -euo pipefail

current_script_path=${BASH_SOURCE[0]}
plugin_dir=$(dirname "$(dirname "$current_script_path")")
# shellcheck source=../lib/utils.bash
source "${plugin_dir}/lib/utils.bash"

list_all_versions
```

---

### 5. Update `bin/latest-stable` (MODERNIZE)

**What:** Keep the core logic but ensure it aligns with asdf 0.17+ expectations.

**Why:** The script is correct but could benefit from better error handling.

**Actions:**
- [ ] Verify it outputs just the version string (no extra whitespace)
- [ ] Add error handling for API failures

**Modern pattern:**
```bash
#!/usr/bin/env bash
set -euo pipefail

current_script_path=${BASH_SOURCE[0]}
plugin_dir=$(dirname "$(dirname "$current_script_path")")
# shellcheck source=../lib/utils.bash
source "${plugin_dir}/lib/utils.bash"

curl_opts=(-sI)

if [ -n "${GITHUB_API_TOKEN:-}" ]; then
    curl_opts=("${curl_opts[@]}" -H "Authorization: token $GITHUB_API_TOKEN")
fi

# curl of REPO/releases/latest is expected to be a 302 to another URL
redirect_url=$(curl "${curl_opts[@]}" "$GH_REPO/releases/latest" | sed -n -e "s|^location: *||p" | sed -n -e "s|\r||p")
version=
printf "redirect url: %s\n" "$redirect_url" >&2
if [[ "$redirect_url" == "$GH_REPO/releases" ]]; then
    version="$(list_all_versions | tail -n1)"
else
    version="$(printf "%s\n" "$redirect_url" | sed 's|.*/tag/v\{0,1\}||')"
fi

printf "%s\n" "$version"
```

**Note:** Removed `sort_versions` since asdf handles sorting now.

---

### 6. Add `bin/test` (OPTIONAL BUT RECOMMENDED)

**What:** Create a `bin/test` script that validates the installation.

**Why:** asdf 0.17+ supports a `bin/test` hook that runs after installation to validate the tool works. This is optional but recommended for better user experience.

**Pattern:**
```bash
#!/usr/bin/env bash
set -euo pipefail

current_script_path=${BASH_SOURCE[0]}
plugin_dir=$(dirname "$(dirname "$current_script_path")")
# shellcheck source=../lib/utils.bash
source "${plugin_dir}/lib/utils.bash"

# Run a simple command to verify the tool works
"$ASDF_INSTALL_PATH/bin/$TOOL_NAME" --version
```

**Actions:**
- [ ] Create `bin/test` that runs `herdr --version`
- [ ] Make it executable
- [ ] Test it manually

---

### 7. Add `bin/info` (OPTIONAL)

**What:** Create a `bin/info` script that shows installation information.

**Why:** asdf 0.17+ supports a `bin/info` hook that displays when running `asdf info herdr`. This is optional but provides useful debugging information.

**Pattern:**
```bash
#!/usr/bin/env bash
set -euo pipefail

echo "herdr version: $(herdr --version 2>/dev/null || echo 'not installed')"
echo "install path: $ASDF_INSTALL_PATH"
echo "download path: $ASDF_DOWNLOAD_PATH"
```

**Actions:**
- [ ] Create `bin/info` (optional, can skip if not needed)

---

### 8. Update Documentation (REQUIRED)

**What:** Update README.md to reflect modern asdf practices.

**Why:** Documentation should match the current state of the plugin.

**Actions:**
- [ ] Update README to mention asdf 0.17+ compatibility
- [ ] Add notes about `plugin.toml`
- [ ] Consider adding a "Contributing" section with modern development practices

---

### 9. Add CI/CD Improvements (RECOMMENDED)

**What:** Update GitHub Actions workflows to test against modern asdf.

**Why:** Ensure the plugin works with the latest asdf binary.

**Actions:**
- [ ] Update `build.yml` to use the latest asdf version
- [ ] Add a test step that runs `bin/test` after installation
- [ ] Consider testing on multiple OS/arch combinations

---

### 10. Consider Additional Features (OPTIONAL)

**What:** Explore optional enhancements.

**Why:** These can improve the user experience.

**Ideas:**
- [ ] Add checksum verification for downloaded binaries
- [ ] Support for multiple binaries (if herdr ships multiple tools)
- [ ] Add `bin/rehash` support (if herdr modifies PATH or other environment)
- [ ] Improve error messages with more context

---

## Implementation Priority

### Phase 1: Required for asdf 0.17+ Compatibility
1. Add `plugin.toml`
2. Remove `sort_versions` from `bin/list-all` and `bin/latest-stable`

### Phase 2: Recommended Improvements
3. Add `bin/test` hook
4. Update documentation
5. Improve error handling in existing scripts

### Phase 3: Optional Enhancements
6. Add `bin/info` hook
7. Add checksum verification
8. Update CI/CD workflows
9. Explore additional features

---

## Testing Plan

After implementing changes:

1. **Manual testing:**
   ```bash
   asdf plugin add herdr /path/to/local/plugin
   asdf install herdr latest
   asdf global herdr latest
   herdr --version
   asdf list herdr
   asdf info herdr
   ```

2. **Automated testing:**
   - Run `bin/test` after install
   - Verify `asdf list-all herdr` outputs sorted versions
   - Verify `asdf latest herdr` returns a valid version

3. **CI testing:**
   - Test on macOS (x86_64 and arm64)
   - Test on Linux (x86_64 and arm64)
   - Test with and without `GITHUB_API_TOKEN`

---

## References

- [asdf documentation](https://asdf-vm.com/)
- [asdf plugin development guide](https://asdf-vm.com/guide/create-a-plugin.html)
- [asdf 0.17 release notes](https://github.com/asdf-vm/asdf/releases/tag/v0.17.0)
- [Modern asdf plugin examples](https://github.com/asdf-vm/asdf/tree/master/lib/commands/plugin)

---

## Next Steps

1. Review this plan
2. Implement Phase 1 changes (required)
3. Test manually
4. Implement Phase 2 changes (recommended)
5. Update documentation
6. Update CI/CD
7. Consider Phase 3 features
