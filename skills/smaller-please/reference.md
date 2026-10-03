# Smaller, Please — verified command reference

All commands below are captured from the shipped `smaller` CLI. Do not invent flags.

## `smaller optimize [OPTIONS] <INPUT>`

Optimize an image, a video, or a directory (a directory is recursively optimized). Sources are
never modified.

Options:

- `--mode <smart|maximum>` — default `smart` (the historic pipeline; Free). `maximum` compresses
  harder and requires the `compression.maximum` capability (Pro). Under Free it is denied with the
  stable `CAPABILITY_REQUIRED` code and nothing is written.
- `--metadata <remove-all>` — the public value is `remove-all` (remove all
  removable metadata, requires the `metadata.control` capability; denied with `CAPABILITY_REQUIRED`
  under Free). With no `--metadata`, Core does **not intentionally strip** metadata: it omits
  explicit removal arguments and leaves handling to the selected backend (some re-encoding
  backends, including the current FFmpeg JPEG path, may naturally omit source EXIF/XMP/IPTC); only
  `remove-all` is an explicit stripping request. `keep`/`remove-privacy` are internal/reserved and
  are not advertised or sellable values.
- `--max-edge` — default `2048`.
- `--quality`
- `--crf`
- `--min-savings`
- `--output` — the default output is the Core managed store path `<bridge_store_dir>/derivatives/optimize/`, and an explicit `--output` overrides it.
- `--force`
- `--dry-run`
- `--json`
- `--image-min-size-mb`
- `--video-min-size-mb`
- `--min-size-mb`

Note: Agents preparing media for AI use `prepare` or `prepare-batch`, not `optimize`.

## `smaller prepare [OPTIONS] <PATH>`

Prepare one image or video for agent input; emits a frozen JSON plan with `use_path`.

Options include `--max-edge`, `--quality`, `--crf`, `--min-savings`, `--output`, `--json`,
`--image-min-size-mb`, `--video-min-size-mb`, `--min-size-mb`.

By default, an optimized derivative is written to the Core managed store's
`<bridge_store_dir>/derivatives/`, and its cache record is written to
`<bridge_store_dir>/cache/`. An explicit `--output <dir>` overrides only the derivative directory;
the cache remains in the managed store. `use_path` is authoritative: always consume the exact
`use_path` from the JSON, and never construct these paths yourself.

### Storage locations (config vs output)

Path setup, listed only to distinguish configuration from derivative storage — do not teach agents
to build these paths manually:

| Location | Purpose |
| --- | --- |
| `~/Library/Application Support/SmallerPlease/config.json` | macOS config file (thresholds and policy). |
| `~/.contextslim-bridge/store` | Default managed store (`bridge_store_dir`). |
| `~/.contextslim-bridge/store/derivatives/...` | Default `prepare` derivative output. |
| `~/.contextslim-bridge/store/cache/...` | Managed cache/dedupe records. |

`~/Library/Application Support/SmallerPlease` is config, not derivative/output storage. A
`prepare` derivative lives in the managed store's `derivatives/`. `use_path` is authoritative.

## `smaller prepare-batch [OPTIONS] <PATHS>...`

Prepare several files or directories in one JSON document. Same options as `prepare`.
The same managed-store default and explicit `--output` behavior apply.

## `smaller doctor [--json]`

Show backend availability.

## `smaller license status [--json]`

Show the local license status (plan, granted capabilities, expiration).

## `smaller license activate <KEY> [--json]`

Verify and store a license key locally (no network). The key is a `SPRO1-…` value; an invalid
key writes nothing.

## `smaller license remove [--json]`

Delete the local license file and return to Free (idempotent).

The Beta.6 release build trusts the Production licence verification key; a valid Production-signed
licence activates Pro. Without a valid licence, the gated flags (`--mode maximum`, `--metadata remove-all`)
are denied with `CAPABILITY_REQUIRED` and nothing is written. The local licence workflow is
`smaller license activate <KEY>` / `status` / `remove` and is fully offline (no network).

## Other top-level commands

- `get skills` — show the bundled Agent Skill content and installation guidance.
- `setup [--json] [--yes]` — set up Core: configuration, managed store, media engine, and native host.
- `repair [--dry-run] [--yes] [--json]` — repair allowlisted installation problems.
- `config show|get|set|reset` — read or update the persisted Core configuration.
- `clean [store|cache|inbox|all] [--dry-run] [--yes] [--json]` — report or remove Smaller, Please-managed cached media.
- `native install|status|uninstall|host` — manage the Chrome Native Messaging host for the browser extension.
- `extension install|status|path|open|uninstall` — stage, inspect, or remove the unpacked Chrome extension bundle.
- `media status|install|path|uninstall` — install, inspect, or remove the optional LGPL Media Pack.
- `uninstall [--incomplete] [--dry-run] [--yes] [--json] [--remove-settings] [--remove-cache]` — remove program/integration files, or clean owned interrupted-install remnants.
