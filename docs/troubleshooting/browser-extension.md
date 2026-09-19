# Troubleshooting: Chrome extension

The extension is an MV3 extension that optimizes images/videos locally before upload on any
normal website (`http://*/*`, `https://*/*`). Core stages the **production bundle** into the
visible `~/Applications/Smaller Please Extension`.

## Commands

```bash
smaller extension status --json   # installed/missing/orphaned/invalid + integrity/recovery state
smaller extension path --json     # the visible Load-unpacked directory
smaller extension install         # stage/update from the resolved bundle
smaller extension open            # macOS: reveal in Finder + open chrome://extensions
smaller extension uninstall       # removes the visible runtime + marker/state
smaller uninstall                 # full program/integration teardown
```

`contextslim extension …` behaves identically (stderr notice only; JSON stdout stays pure).

## Staged, loaded, and connected are different

`smaller extension status` / `doctor`'s `browser_integration` report three separate facts:

- **staged** — the verified bundle is in `~/Applications/Smaller Please Extension`
  (`bundle_installed`);
- **loaded** — you enabled Developer mode and clicked Load unpacked. Core cannot observe
  this, so `load_state` is always `"unknown"` (`requires_user_confirmation`);
- **connected** — a live Native Messaging handshake (`native_host` check), not the bundle.

Never read `installed: "installed"` as "the extension is running in Chrome".

## Bundle integrity and validation

`smaller extension install` validates the bundle (schema, product, MV3, pinned id, file hashes)
before writing. It rejects a symlinked bundle, missing/extra/modified files, a wrong extension
id, and malformed metadata, leaving the previous valid install intact. The bundle metadata is
**integrity/ownership metadata, not a signature**.

`smaller extension status` reports `integrity: invalid` when the staged tree no longer
matches its marker (e.g. a manually edited file). Re-run `smaller extension install` or
`smaller repair` to restore it.

## Extension fails to load / "Manifest file is missing or unreadable"

You probably selected the wrong directory. Select the staged directory printed by
`smaller extension path` (`~/Applications/Smaller Please Extension`, the one containing
`manifest.json`).

## Developer mode / Load unpacked

Chrome requires **Developer mode** to be on before **Load unpacked** is available. This step
cannot be automated; never point automation at your real Chrome profile.

`smaller setup` stages the bundle and prints these manual steps (open `chrome://extensions`,
enable Developer mode, Load unpacked, select `~/Applications/Smaller Please Extension`). It never
loads or registers the extension and never modifies the real Chrome profile. `smaller extension
open` only reveals the directory and opens `chrome://extensions`.

`smaller doctor`'s `browser_integration` check is deliberately honest and separates four
states:

- `extension_detected` (`✓ Ready`) — the staged bundle is present and valid **and** the native
  host manifest allows the pinned extension id;
- `extension_manual_setup_required` (`! Waiting for manual Chrome setup`) — the bundle files are
  prepared but the one-time Chrome **Developer mode → Load unpacked** step is still required;
- `extension_not_detected` (`! Not installed`) — no bundle and no registration were found;
- `extension_unverified` — a registration manifest exists but its allowed origin could not be
  verified.

Its `details` additively report `bundle_installed`, `bundle_valid`, `bundle_state`,
`extension_version`, and `load_state`; it always reports `enabled: "unknown"` — the CLI cannot
confirm a real Chrome profile, so it never claims the extension is installed or enabled.

### Orphaned extension state (files removed, Chrome still lists it)

If the visible runtime (`~/Applications/Smaller Please Extension`) is removed while the
ownership metadata remains, `smaller extension status` reports `installed: "orphaned"` and
`doctor` reports `extension_orphaned` (`! Removed locally — remove it in Chrome`). Chrome keeps
unpacked extensions after their files disappear, so open `chrome://extensions`, find
Smaller Please, and click **Remove**. This is a warning, never an error, and `load_state`
stays honestly `unknown`. Re-run `smaller extension install` (or `smaller repair --yes`) to
re-stage the bundle instead if you want to keep using it. An orphaned state does not change
what `smaller uninstall` deletes.

## Extension reported "invalid" after a manual edit

Do not hand-edit the staged tree. `smaller extension install` (or `smaller repair --yes`)
re-stages the verified bundle atomically; `smaller extension uninstall` removes the managed
visible runtime plus the marker/state and never touches source files.

## CSP error / a page appears stuck

Extension pages run under `script-src 'self'`; all script lives in bundled `dist/*.js` files.
A CSP violation usually means a stale build. Re-run the installer or `smaller repair`, then
reload the extension.

## Popup shows "Core settings unavailable"

That is a Core/Native Host problem, not an extension problem. See
[`native-host.md`](native-host.md). The popup writes a stable
`smaller stage=<stage> code=<code>` line to its own console; raw native/Core prose is never
rendered.

## Popup/install page shows `Core still not reachable`

The install page's `Retry connection` re-runs the handshake. Verify:

```bash
smaller doctor
smaller native status
smaller repair          # reinstalls the allowlisted native registration and re-handshakes
```

Then reload the extension.

## "Core <version> ready" but the browser still behaves like an old build

The installed Core, the native host launcher, and the extension files may have drifted. Refresh
them with:

```bash
smaller repair
```

Then reload the extension and hard-refresh the site. If it persists, re-run the latest installer
so Core, the extension bundle, and the launcher come from the same release.

## Language does not change in an open website tab

The extension now watches `chrome.storage.onChanged` for `contextslim_locale_v1`, so a popup
language change applies to the next in-page chip/progress without a page refresh. If an
old tab still shows the previous language, reload the extension once so the new content
script is injected, then hard-refresh the site. A language change never triggers an
optimization, a native request, or a history/metrics write.

## Permissions / privacy

The manifest requests exactly `nativeMessaging` + `storage`, and site access for normal websites
(`http://*/*`, `https://*/*`) so an image/video can be detected before it uploads. There is no
`webRequest`/`webRequestBlocking`, no `scripting`, and no `web_accessible_resources`. The
extension does not read page content, chat text, cookies, or your browsing history. Media never
leaves the device.

## Uninstall

For a full teardown, use the top-level lifecycle; it removes the visible runtime, the Native
Host, the managed Core, and the managed Media Pack, while keeping config, store/cache, and
browser-local data:

```bash
smaller uninstall --dry-run --json   # zero mutation, shows the plan
smaller uninstall                    # interactive confirmation
smaller uninstall --remove-settings --remove-cache --yes   # optional fresh-install behavior
```

For the extension files only:

```bash
smaller extension uninstall --dry-run   # zero mutation, shows what would be removed
smaller extension uninstall --yes       # removes the visible runtime + marker/state
```

Then open `chrome://extensions` and click **Remove** on the unpacked entry — Chrome does not
allow a program to remove an unpacked extension from the real profile, so that step is always
manual (and it is the only way to clear the `contextslim_*` `chrome.storage`). `extension
uninstall` never touches config, the store, `chrome.storage`, or the Chrome profile. See
[`../uninstall.md`](../uninstall.md) for the full sequence, the installer GUI sheet, and the
Homebrew boundary.
