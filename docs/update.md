# Updating Smaller Please

> This page separates what works today (**Implemented**) from the designed updater
> (**Planned**). Do not script the planned commands; they do not exist yet.

## Version domains

| Domain | Today | Check |
|---|---|---|
| Extension version | `0.1.0` | `chrome://extensions` |
| Core version | `0.1.0` | `smaller --version` |
| Native protocol version | `1` | handshake |
| Media Pack version | `7.1-lgpl.1` | `smaller media status --json` |

## Compatibility rule

**"Extension requires Core >= X."** A new extension must fail against a stale Core with a
clear, actionable message, never an opaque runtime error.

- **Implemented today:** the wire envelope checks `protocol_version` and returns a framed
  `bad_protocol_version`, which the UI renders as a localized message (raw Core text is never
  shown).
- **Planned:** a semantic Core-version floor (compare the handshake `core_version` against the
  extension's minimum) so an older-but-protocol-compatible Core is reported precisely.

Additive fields within protocol/schema v1 are backward compatible: an old host ignores new
optional fields (for example `image_min_bytes` / `video_min_bytes`), and an old client gets
Core defaults for omitted fields.

## Update today (installer distribution)

Update by running the newer installer, then reloading the extension:

1. Download and open the newer `Smaller Please Installer.app` and click **Install**.
2. The installer verifies its embedded payload, compares versions, atomically replaces the
   managed Core, stages the newer extension bundle (the fixed Load-unpacked path never changes),
   updates the Media Pack only if needed, then runs `setup`/`repair` and `doctor`.
3. Open `chrome://extensions` and click **Reload** on the Smaller Please card, then hard-refresh
   the target site tab. Chrome is never closed for you.

A same-version reinstall is a no-op and preserves `config.json`, the managed store, and the
staged extension. There is **no background auto-update** (`smaller update` and automatic update
distribution are **Planned**).

### Staging the extension explicitly

For an installed distribution the payload is resolved automatically:

```bash
smaller extension install     # same fingerprint -> no-op; else atomic update
smaller extension status --json
```

`smaller setup` and `smaller repair` reuse the same staging step. Re-staging an unchanged
bundle is a no-op, so re-running is safe and idempotent.

### Homebrew install

A **public** Homebrew tap and a signed/notarized Homebrew release are **Planned**; there is no
public tap today, so there is no Homebrew upgrade command to document yet. See
[`docs/install/homebrew.md`](install/homebrew.md).

### Media Pack

The optional LGPL Media Pack is a **separate artifact** from Core/extension. Update by installing
the newer local artifact; the previous valid version is retained as a rollback target and older
managed versions are cleaned up:

```bash
smaller media install --source /path/to/smaller-media-<new-version>-macos-arm64.tar.gz
smaller media status --json     # version, integrity, current
smaller media path --json       # the active bin/
```

A same-version install of a valid, current pack is a no-op. Any failure leaves the previous
working version and pointer intact. `--dry-run` changes nothing. There is **no online
downloader**; a public, signed/notarized download service is **Planned**. A Homebrew/system
backend still wins over an installed pack, so updating the pack never downgrades a healthy
backend.

### Core health after an update

```bash
smaller doctor     # read-only health report (all six checks)
smaller repair     # allowlisted fixes if native/config/store drifted
```

`smaller setup` is the first-install path and is also safe to re-run after an update, but it is
not the updater itself. `launcher current: true` means only that the native launcher equals the
installed CLI; after a Core update, re-run `smaller repair` (or `smaller native install`) so the
launcher matches.

## Update (planned)

- `smaller update` — a single Core-driven update for unpacked installs: refresh the fixed
  extension directory from a verified source, refresh the launcher, prompt a reload.
- A signed/notarized updater that replaces the app and Core is **Planned**.
- **Chrome Web Store:** once listed, Chrome auto-updates the extension; only Core/Native/Media
  remain installer responsibilities.
- **Homebrew users:** a **public** `homebrew-smaller-please` tap and signed/notarized upgrade are
  **Planned**.
- **Media Pack:** the local versioned install/update/rollback/uninstall **is Implemented**
  (`smaller media …`, never an unverified mirror). A public download plus signature/notarization
  is **Planned**.

## Update safety rules

- Never modify the real Chrome profile contents.
- Never auto-close Chrome.
- Never delete user data during an update.
- HTTPS-only, checksum/signature-verified payloads; no unknown mirrors.
