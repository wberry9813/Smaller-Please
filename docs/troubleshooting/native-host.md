# Troubleshooting: Native Host

The Chrome native host is named `com.contextslim.bridge` (a stable protocol identifier, not
branding). The extension talks to the local Core through it; there is no server and no network
traffic.

## Inspect the install

```bash
smaller doctor --json          # native_host check: status/code + manifest/launcher details
smaller native status
smaller native status --json
```

`doctor`'s `native_host` check reports one of: `native_ok`, `native_manifest_missing`,
`native_launcher_missing`, `native_launcher_stale`, `native_origin_stale`,
`native_handshake_failed` (with `manifest_path`, `launcher_path`, `launcher_matches_current`,
`extension_id`, and — on success — `protocol_version`/`core_version`).

`native status` JSON keys: `platform`, `browser`, `host_name`, `manifest_path`,
`manifest_present`, `launcher_path`, `launcher_present`, `launcher_matches_current`,
`extension_id`, `binary_path`.

Paths:

| Item | Path |
|---|---|
| Launcher | `~/.contextslim-bridge/native/contextslim-native-host` |
| Chrome host manifest | `~/Library/Application Support/Google/Chrome/NativeMessagingHosts/com.contextslim.bridge.json` |

## `manifest_present: false` (`native_manifest_missing`)

The Chrome host manifest is missing. Register it:

```bash
smaller native install
smaller native status
```

`smaller repair` reinstalls the manifest and launcher automatically (then re-runs the
handshake) if you want the allowlisted fix. `native install` is idempotent, needs no `sudo`,
and never touches other hosts.

## `launcher_present: false` / `launcher current: false` (`native_launcher_missing` / `native_launcher_stale`)

The launcher is missing or does not match the CLI that ran `native status`. Re-run:

```bash
smaller native install     # or: smaller repair
```

`launcher current: true` means only that the launcher equals the CLI that ran the command. It
does **not** mean the installed CLI equals the newest release. After updating Core, re-run
`smaller repair` (or `smaller native install`) so the launcher matches.

## `native_origin_stale` / `native_handshake_failed`

`native_origin_stale` means the manifest's `allowed_origins` does not contain the pinned
extension id; `native_handshake_failed` means the real framed handshake did not return a valid
`handshake` result. Both are auto-repairable:

```bash
smaller repair
```

`repair` reinstalls the manifest + launcher and confirms the handshake. If it still fails,
the installed launcher is likely out of sync with the extension — re-run `smaller repair`, then
reload the extension.

## The extension says "Core settings unavailable"

The popup logs a stable `smaller stage=config-get code=<code>` code to its own console.
Common codes and meaning:

| Code | Meaning |
|---|---|
| `native-connect-failed` | Chrome could not start the host. Check `manifest_present`, `launcher_present`, and that the manifest `path` exists and is executable. |
| `native-request-failed` | The host connected but did not answer in time. |
| `native-rejected` | Core answered with an error. |
| `invalid-config` | Core returned an unreadable configuration. |
| `service-worker-unreachable` / `service-worker-rejected` | Reload the extension at `chrome://extensions`. |

Raw host messages are never rendered in the UI; the code is the diagnostic.

## Chrome reports "Native host has exited"

The host process exited at launch. The most common historical cause was an unreadable store;
Core now degrades to a store-unavailable state instead of exiting, and store-requiring
operations return a framed `store_unavailable` error. Re-run `smaller native install`,
reload the extension, and check [`storage.md`](storage.md).

## Extension id mismatch

The manifest's `allowed_origins` must contain the extension's real id. `native status`
reports the id read from the manifest, or the pinned default when the manifest is absent. To
install for a different id:

```bash
smaller native install --extension-id <32-char-id>
```

Chrome extension ids are exactly 32 characters in `a..p`; anything else is rejected.

## Refresh the launcher after an update

If the browser behaves like an older build after an update, the native host launcher is often the
stale copy. Run:

```bash
smaller repair
```

It reinstalls the manifest and launcher and re-runs the handshake. It never deletes user data and
never uses `sudo`. Then reload the unpacked extension and hard-refresh the site.

## Repair vs. reinstall

`smaller repair` performs the allowlisted native fix (reinstall manifest + launcher, then
re-run the handshake) and reports the final status. It never touches unrelated host manifests
or the real Chrome profile. `smaller native install` remains the explicit one-shot command.

## Extension staging

Core stages the verified production bundle to the visible
`~/Applications/Smaller Please Extension` (`smaller extension install`, `setup`, `repair`), with
the ownership marker/state under `~/Library/Application Support/SmallerPlease/extension/`.
Loading it stays a manual `chrome://extensions` **Developer mode / Load unpacked** step; see
[`browser-extension.md`](browser-extension.md).
