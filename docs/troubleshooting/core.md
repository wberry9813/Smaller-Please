# Troubleshooting: Core (`smaller`)

## First checks

```bash
which smaller
smaller --version
smaller doctor --json
```

Expected: the binary resolves on `PATH`, prints `smaller 0.1.0`, and `doctor` reports
`overall_status` with the six checks (`core`, `config`, `storage`, `media_engine`,
`native_host`, `browser_integration`). The legacy `contextslim` command is still installed and
works as an alias.

- `smaller setup` ensures config, the managed store, the Media Engine, and the Native Host
  (idempotent; `--yes` never prompts and never installs Homebrew).
- `smaller repair --dry-run` previews the allowlisted fixes; `smaller repair` applies them and
  re-runs the handshake and `doctor`.

## `command not found: smaller`

The CLI is not on `PATH`. If you installed with the app, the managed binaries live at
`~/Library/Application Support/SmallerPlease/bin/` (`smaller` and the `contextslim` alias). Add
that directory to your `PATH`, or run the binary by full path, then re-run.

## A new flag or subcommand is rejected

The installed CLI is older than the release you are following. Re-run the latest installer, then
`smaller repair` and reload the extension. See [`../update.md`](../update.md).

## Config problems

Config lives at `~/Library/Application Support/SmallerPlease/config.json`
(`$XDG_CONFIG_HOME/smaller-please/config.json` or `~/.config/smaller-please/config.json`
elsewhere; `SMALLER_CONFIG_DIR` overrides the directory for portable setups). The
legacy `CONTEXTSLIM_CONFIG_DIR` is still honored, and an existing legacy
`ContextSlim/config.json` is migrated to the canonical path on first load (the legacy file is
left untouched).

```bash
smaller config show --json
smaller config get image-min-size-mb
smaller config set image-min-size-mb 6
smaller config reset --all
```

- A corrupt file recovers to defaults with a diagnostic on stderr; `config show` still works.
- Unknown future fields are preserved across a read-modify-write.
- Writes are atomic (same-directory temp + rename).
- `config_set` rejections never partially apply; an invalid value leaves the previous value.

## `doctor` statuses

`doctor` is read-only and its JSON keys are the stable machine-readable surface. It keeps the
original tool keys and adds the health model (`schema_version: 1`):

```json
{
  "schema_version": 1,
  "overall_status": "warning",
  "brand": "Smaller Please",
  "core_version": "0.1.0",
  "versions": { "core_version": "0.1.0", "native_protocol_version": 1,
                "bridge_schema_version": 1, "extension_version": null,
                "media_pack_version": null },
  "checks": [
    { "id": "core", "status": "ok", "code": "core_ok", "details": {}, "message": null, "fix": null },
    { "id": "config", "status": "ok", "code": "config_ok", "details": {}, "message": null, "fix": null },
    { "id": "storage", "status": "ok", "code": "store_ok", "details": {}, "message": null, "fix": null },
    { "id": "media_engine", "status": "ok", "code": "media_engine_ok", "details": {}, "message": null, "fix": null },
    { "id": "native_host", "status": "ok", "code": "native_ok", "details": {}, "message": null, "fix": null },
    { "id": "browser_integration", "status": "warning", "code": "extension_not_detected", "details": {}, "message": null, "fix": {} }
  ],
  "platform": "macos",
  "ffmpeg":  { "status": "ok", "detail": null },
  "ffprobe": { "status": "ok", "detail": null },
  "sips":    { "status": "ok", "detail": null }
}
```

- `checks[].status` is `ok | warning | error | unavailable | not_configured`; each check has a
  stable `code` and a `details` object. `overall_status` is `error` if any check errors, else
  `warning` if any warns, else `ok`.
- `core`: `core_ok` / `legacy_cli_missing` (the `contextslim` alias is not on `PATH`) /
  `core_version_unavailable`.
- `config`: `config_ok` / `config_missing` / `config_invalid` / `config_migration_pending` /
  `config_dir_unwritable`. `doctor` is read-only, so a pending migration is reported here as
  `config_migration_pending` with `migration_pending: true`. Only `smaller setup`/`smaller
  repair` perform the migration.
- `storage`: `store_ok` / `store_missing` / `store_marker_missing` / `store_legacy_adoptable` /
  `store_not_managed` / `store_unavailable` (see [`storage.md`](storage.md)).
- `media_engine`: `media_engine_ok` / `media_backend_missing` / `media_capability_missing`
  (see [`media-engine.md`](media-engine.md)).
- `native_host`: `native_ok` / `native_manifest_missing` / `native_launcher_missing` /
  `native_launcher_stale` / `native_origin_stale` / `native_handshake_failed`
  (see [`native-host.md`](native-host.md)).
- `browser_integration`: `extension_detected` (bundle ready + pinned-id registration) /
  `extension_manual_setup_required` (bundle staged; manual Chrome step pending) /
  `extension_not_detected` / `extension_unverified` / `extension_orphaned` (files removed but
  Chrome may still list it; remove it manually) (see [`browser-extension.md`](browser-extension.md)).
- Legacy `ffmpeg`/`ffprobe` status is `ok`, `not_found`, or `failed_to_launch`; `sips`
  additionally reports `not_applicable` on non-macOS. `detail` carries the first stderr line or
  an exit code, never a media path.

`doctor` never migrates config, adopts a store, or write-probes a directory; it exits `0`.
Most non-`ok` checks map to an allowlisted `smaller repair` action, and `setup` covers the
first-install path.

## Where data lives (and does not)

- Config: `~/Library/Application Support/SmallerPlease/config.json` (never auto-deleted;
  legacy `ContextSlim/config.json` is migrated, not deleted).
- Managed store: `bridge_store_dir`, default `~/.contextslim-bridge/store` — see
  [`storage.md`](storage.md).
- Browser-local history/metrics: in `chrome.storage.local`, never touched by Core.

## Removing the CLI

Use `smaller uninstall` (see [`../uninstall.md`](../uninstall.md)); it removes the installed
program/integration files with ownership checks and, by default, keeps your config and managed
store. Removing only the binaries does not remove the native host, the config, or the store.
