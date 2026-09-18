# Troubleshooting: storage and `clean`

Smaller, Please stores only generated derivatives/cache. **Source files are never stored,
modified, or deleted.**

## Where storage lives

| Item | Path |
|---|---|
| Managed store (`bridge_store_dir`) | default `~/.contextslim-bridge/store` |
| Store marker | `<store>/.contextslim-store.json` |
| Derivatives | `<store>/derivatives/` |
| Core cache | `<store>/cache/` |
| Staging tmp | `<store>/tmp/` |
| Bridge inbox | `~/.contextslim-bridge/inbox/` |

The store is a **marker-validated managed directory**. `clean` may only remove managed
content plus the bridge-owned `inbox/`; it never follows a symlinked managed subdirectory out
of the store and never removes user files, the config file, or browser-local history/metrics.

## Diagnose and repair the store

```bash
smaller doctor --json     # storage check
smaller setup             # create the managed layout + marker (or adopt the legacy default)
smaller repair            # same allowlisted fixes, plus a dry-run preview
```

The `storage` check reports one of: `store_ok`, `store_missing`, `store_legacy_adoptable`,
`store_marker_missing`, `store_not_managed`, `store_unavailable`. `setup`/`repair` only
create the managed layout + marker or adopt the exact valid legacy default; an arbitrary
unmarked non-empty directory becomes a user action and is never adopted or deleted.

## Inspect and clean

```bash
smaller clean --dry-run --json
smaller clean --dry-run
smaller clean --yes
smaller clean cache --yes
smaller clean inbox --yes
```

- `--dry-run` reports reclaimable bytes and modifies nothing.
- A non-interactive run without `--yes` refuses.
- The popup's Storage panel reads the same numbers (native `storage_stats`), so the popup and
  the CLI agree.

## Change the store location

```bash
smaller config set store-dir ~/SmallerPleaseStore
smaller config get store-dir
```

Changing the location does **not** migrate or delete old content. New derivatives/cache go to
the new store; clean the old store separately. `store-dir` is validated (tilde expansion,
absolute normalization, a write probe, and rejection of dangerous roots such as `/`, `$HOME`,
`/Users`, `/System`, `/tmp`, `/var`, `/etc`, and symlinks into them); an invalid value is
rejected and the previous value is kept.

## Unmarked directory / `store_not_managed` / `store_unavailable`

A directory that is not marker-validated is refused unless it is the exact legacy default
`~/.contextslim-bridge/store` and its contents validate as pre-existing ContextSlim output.
This is deliberate: it prevents `clean`/`repair` from adopting and deleting pre-existing user
content. `doctor` reports `store_marker_missing` when a marker exists but is invalid, and
`store_not_managed` for an arbitrary directory; `setup`/`repair` can safely rewrite a marker
or create the layout only when the directory holds nothing but the empty managed layout.
If the native bridge reports `store_unavailable`, the host stays alive and store-requiring
operations return a framed error; fix `bridge_store_dir` with
`smaller config set store-dir <path>` or run `smaller clean --dry-run` to inspect.

## Why `.contextslim` is special

The CLI's default output directory is `.contextslim/` beside a source. It is generated output
and is **never** scanned as input by `batch`/`prepare-batch`. Cleaning it is a manual
filesystem action; `smaller clean` targets the managed store, not arbitrary `.contextslim`
directories.

## Uninstall

`smaller uninstall` handles the program/integration teardown and keeps cached media by default;
`--remove-cache` is the explicit opt-in that removes only marker-proven managed store/cache
(`clean` remains the scoped cache/store command). An unprovable custom `store-dir` is refused,
never deleted. See [`../uninstall.md`](../uninstall.md).
