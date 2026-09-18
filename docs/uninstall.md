# Uninstalling Smaller, Please

> **Program components** and **user data** are separate. `smaller uninstall` removes the
> program/integration by default and never deletes user data; it only touches settings or cached
> media when you explicitly pass `--remove-settings` / `--remove-cache`. It never closes Chrome
> or modifies a Chrome profile.

## What is installed

Program components:

- the Core CLI binaries (`smaller`, plus the `contextslim` compatibility alias), normally linked
  into `<prefix>/bin`;
- the **installer-managed** Core binaries at
  `~/Library/Application Support/SmallerPlease/bin/{smaller,contextslim}` with the ownership
  marker `bin/.smaller-core.json` (created by the installer app, independent of any Homebrew
  copy);
- the installer app itself (`Smaller Please.app`), wherever the user put it;
- the native host launcher `~/.contextslim-bridge/native/contextslim-native-host` (an independent
  copy of the binary, never a symlink into a Homebrew Cellar);
- the Chrome host manifest
  `~/Library/Application Support/Google/Chrome/NativeMessagingHosts/com.contextslim.bridge.json`;
- the visible staged unpacked extension `~/Applications/Smaller Please Extension/` and its
  ownership metadata `~/Library/Application Support/SmallerPlease/extension/`;
- the optional LGPL Media Pack under `~/Library/Application Support/SmallerPlease/media/`
  (only when installed with `smaller media install`; the prompt to install it comes from
  `smaller setup`).

User data:

- Core config `~/Library/Application Support/SmallerPlease/config.json` (a legacy
  `ContextSlim/config.json` is migrated on first load and left in place);
- the managed store (`bridge_store_dir`, default `~/.contextslim-bridge/store`);
- browser-local history/metrics in `chrome.storage.local` (the `contextslim_*` keys).

## `smaller uninstall` (Implemented)

```bash
smaller uninstall [--incomplete] [--dry-run] [--yes] [--json] \
                  [--remove-settings] [--remove-cache]
```

| Flag / mode | Effect |
|---|---|
| *(none)* | **Default:** remove program/integration — the installer-managed Core `bin/`, the Native Host manifest + launcher, the visible extension, and the managed Media Pack. Keep config, store/cache, browser-local data, a Homebrew package, an external/system FFmpeg, and user media. |
| `--remove-settings` | Also remove the canonical config (and a provably-owned legacy config). Never the whole Application Support root. |
| `--remove-cache` | Also remove the marker-proven managed store/cache. An unprovable custom `store-dir` is **refused**, never deleted. |
| `--incomplete` | **Clean Incomplete Installation:** remove only owned interrupted-install remnants (see below). Never a valid install, config, store/cache, browser storage, Homebrew, or unknown file. |
| `--dry-run` | Print the plan; byte/mtime **zero mutation**. |
| `--yes` | Skip the interactive `[y/N]` confirmation. Required in a non-interactive shell. |
| `--json` | Exactly one machine-readable document on stdout (schema `1`); diagnostics go to stderr. |

`contextslim uninstall …` behaves identically (the legacy notice goes to stderr only, so JSON
stdout stays pure).

### Safety and ownership

- Every deletion is **ownership-proven**. Tampered, symlinked, foreign, or unknown content is
  refused and reported in `remaining_user_actions` — a refusal is a skip, never a wider
  deletion.
- **Core self-removal:** the managed Core is removed **last**, so standalone
  `smaller uninstall --yes` can unlink its own running binary's directory and exit cleanly. It
  never requires the GUI.
- The Media Pack is removed only through its ownership/integrity gate; a Homebrew, system, or
  custom `ffmpeg` is never touched. `/Applications`, `/opt/homebrew/Cellar`,
  `/usr/local/Cellar`, brew links, and the real Chrome profile are never touched.
- A per-item failure (`performed_actions[].status == "failed"`) never widens deletion, and a
  second run is a safe no-op (`already_absent`).

### JSON contract (schema 1)

```json
{
  "schema_version": 1,
  "mode": "uninstall",
  "initial_state": { "core": {}, "native_host": {}, "extension": {}, "media": {}, "settings": {}, "cache": {}, "homebrew": {}, "incomplete": {} },
  "planned_actions": [{"id": "extension_full", "component": "extension", "status": "planned", "code": "extension_remove"}],
  "performed_actions": [{"id": "core_bin", "component": "core", "status": "performed", "code": "core_remove"}],
  "kept_data": [{"id": "config", "component": "settings", "code": "settings_kept"}],
  "remaining_user_actions": [{"id": "chrome_remove", "component": "extension", "code": "chrome_remove_manual", "message": "Open chrome://extensions and click Remove…"}],
  "overall_status": "ok"
}
```

`mode` is `uninstall` or `incomplete_cleanup`. Exit `0` for success or an ownership
refusal-warning; exit `2` when an automatic action failed or a non-interactive run lacked
`--yes` (`confirmation_required`).

A declined interactive confirmation emits the read-only plan and mutates nothing.

## Clean an incomplete installation (`--incomplete`)

If an install or update is interrupted, the installer detects owned incomplete state from actual
staging/backups/markers and offers **Clean Incomplete Installation**. The same behavior is
available from the CLI:

```bash
smaller uninstall --incomplete --dry-run --json   # read-only plan
smaller uninstall --incomplete --yes              # clean only owned remnants
```

It removes only owned remnants: Core extraction/staging, `.bin-staging-*`, `.bin-backup-*`, a
partial markerless `bin/`, an ownership-proven `.install.lock`, a partial/orphaned Native Host
manifest or launcher, extension staging/backups plus an unbound partial visible runtime
(interrupted activation), and Media Pack staging/incomplete versions. A valid installed tree,
config, store/cache, browser storage, Homebrew, and unknown files are always kept.

A leftover `.install.lock` is harmless — the kernel releases the advisory `flock` when the
process exits, so it never blocks a reinstall; the installer GUI therefore filters it out of the
"needs cleanup" banner, while the CLI still lists/removes it as an owned remnant.

## Installer app: the `Uninstall…` sheet

The installer app has an **`Uninstall…`** action. It opens a confirmation sheet that lists what
will be removed (Smaller, Please Core, Media Engine, Browser integration, extension files) and
what is kept (settings, cached/optimized media, browser-local history). **`Keep settings`** and
**`Keep cached media`** default **ON**; confirming calls the Core uninstall JSON lifecycle with
the corresponding `--remove-settings`/`--remove-cache` flags only when a box is unchecked.
Cancelling is a zero-mutation path.

The sheet also explains that the `contextslim_*` Chrome storage is only removable in Chrome via
**Remove**, and that a Homebrew installation is kept (`brew uninstall smaller-please` is
suggested, never run by the GUI). After uninstall the app refreshes to **Not installed**
(connection **Not configured**) and shows the retained settings/cache separately — not as
corruption. The installer app bundle itself is removed by dragging `Smaller Please.app`
to the Trash.

## Chrome extension: the final manual `Remove`

Open `chrome://extensions`, find Smaller, Please, and click **Remove**.

This is always a manual step — Chrome does not allow a program to remove an unpacked extension
from the real profile, and Smaller, Please never closes Chrome. `smaller extension open` (or the
installer's **Open Chrome Extensions**) reveals the directory and opens the page. Removing the
unpacked entry is what clears the extension's `chrome.storage.local` (`contextslim_*` keys);
neither the CLI nor the installer can do that.

If the extension files have already been removed from disk but Chrome still lists the entry,
`smaller extension status` reports `installed: "orphaned"` and `smaller doctor` reports
`extension_orphaned` (`! Removed locally — remove it in Chrome`). The **Remove** click in
`chrome://extensions` is still required; the orphaned state is a warning, not an error, and says
nothing about whether Chrome currently has the extension loaded.

## Homebrew boundary

If you installed via Homebrew, the CLI does not remove the Homebrew package:

```bash
brew uninstall smaller-please
```

This removes **only Homebrew-managed payload** (the Cellar tree plus the linked `bin/` and
`share/` files). It does **not** remove the user-level staged extension
`~/Applications/Smaller Please Extension/`, its metadata, the Native Host launcher/manifest,
config, store/cache, or browser-local data. Run `smaller uninstall` for those. `smaller
uninstall` never runs `brew` for you. (A **public** Homebrew tap is **Planned**; see
[`install/homebrew.md`](install/homebrew.md).)

## What `smaller uninstall` does not touch

- Config and store/cache (unless `--remove-settings` / `--remove-cache`).
- Browser-local history/metrics (`chrome.storage.local`).
- An external/system/Homebrew `ffmpeg` + `ffprobe`.
- The Homebrew package itself.
- Your media files (sources are never modified by Smaller, Please).
- Other Chrome Native Messaging hosts or anything outside the managed paths above.

## After uninstall / reinstall

`smaller doctor` reports the program components as not installed. Reinstall with the installer
or `smaller setup`; retained settings and cached media are reused. For fresh-install behavior:

```bash
smaller uninstall --remove-settings --remove-cache --yes
```

## Manual / granular steps (fallback)

For a Homebrew install, or for fine-grained control, the individual commands still work:

1. Remove the staged extension files:
   ```bash
   smaller extension uninstall --dry-run   # zero mutation
   smaller extension uninstall --yes
   ```
   This removes the visible runtime plus marker/state after the ownership/integrity check.
2. Click **Remove** in `chrome://extensions` (always manual).
3. Unregister the native host: `smaller native uninstall`.
4. Optionally remove the Media Pack:
   ```bash
   smaller media status --json
   smaller media uninstall --dry-run
   smaller media uninstall --yes
   ```
5. Optionally remove the installer-managed Core `bin/` — prefer `smaller uninstall`, which does
   this with ownership checks.
6. Optionally reclaim cached media while the CLI is still installed:
   ```bash
   smaller clean --dry-run --json
   smaller clean --yes
   ```
   `clean` only removes marker-validated managed content plus the bridge-owned `inbox/`; it never
   touches source files, the config file, or browser-local history/metrics.
