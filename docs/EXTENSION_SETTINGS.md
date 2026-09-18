# Extension Settings and Website Rules

Status: **Implemented**.

This document describes the Chrome extension's browser-local **compression control**: the one
global switch, the two website-policy modes, the unified Website Rules list, the supported
file-type gate, and the export/import configuration format.

It is deliberately **not** a Core config document. Optimization thresholds (image/video minimum
size, minimum savings, store location) are owned by Smaller, Please Core/CLI and are read by the
extension through `config_get`. See [`CLI.md`](CLI.md).

## 1. Ownership and storage

The control document is **browser-local**, like the locale and history. It is persisted in
`chrome.storage.local` under the single versioned key:

```
contextslim_control_v1
```

The extension does **not** write any threshold copy here, and it does not reuse the existing
`contextslim_locale_v1` / `_history_v1` / `_metrics_v1` / `_settings_v1` keys. There is no CLI
or Native-Host management of this document.

### Schema

```json
{
  "control_schema_version": 1,
  "enabled": true,
  "mode": "allowlist",
  "siteRules": [{ "domain": "chatgpt.com", "enabled": true }]
}
```

This is the **fresh-install default shape**: the switch is on, the mode is the AI allowlist, and
the built-in AI domains are present and enabled (the example shows one; the real default has all
nine — see §5).

| Field | Type | Meaning |
|---|---|---|
| `control_schema_version` | `1` | Schema version. Any other value (unknown/future) is treated as a corrupt document. |
| `enabled` | boolean | **The one global switch and the source of truth for on/off.** `false` blocks interception everywhere and preserves `mode` and every `siteRules` entry. It is stored and read as-is; it is never derived from `mode`. |
| `mode` | `"all"` \| `"allowlist"` | The website policy. A **fresh/corrupt** document defaults to `"allowlist"`; a document that already has a usable `mode` keeps it. There is no stored `"disabled"` mode. |
| `siteRules` | array | `{ domain, enabled }` rules, deduplicated and capped. A **fresh/corrupt** document contains every built-in AI domain enabled; an existing document keeps its stored rules. |

The schema version stays `1`: this is the same v1 shape with a reduced value set and `enabled`
promoted to the global truth. `pausedUntil` is gone; the old 30-minute pause does not exist.

## 2. The global switch

The Settings page **General** section and the popup both expose the same single switch
(`enabled`):

- `enabled: true` — Smaller, Please may intercept supported uploads according to the website
  policy below.
- `enabled: false` — Smaller, Please never intercepts an upload anywhere; **`mode` and all
  `siteRules` are kept exactly as they were**.

Toggling the switch only changes `enabled`. It can never change `mode` or a rule, so turning
Smaller, Please back on restores the previous configuration untouched. There is no separate
pause state.

## 3. Modes and the exact decision

| Mode (`mode`) | Settings label | Meaning |
|---|---|---|
| `"all"` | Optimize on all websites | Every website where the extension runs is optimized unless a rule turns that website off. |
| `"allowlist"` | Only selected websites | Only websites with an **explicit enabled rule** are optimized. A website with no rule is **not** optimized. This is the fresh-install default (with all built-in AI sites enabled). |

The effective decision for a page host is, in order:

```
enabled === false     -> disabled
mode === "allowlist"  -> rule ? rule.enabled : disabled
mode === "all"        -> rule ? rule.enabled : enabled
```

i.e. **the global switch > a site rule > the mode default**. `"disabled"` is not a mode; to
turn everything off, use the switch.

When compression is off for a page, the extension does **not** intercept the upload at all: no
`preventDefault`/`stopImmediatePropagation`, no chip, no native call, and no metrics/history
entry. The website's own upload path runs untouched.

### Sanitize-on-read (never throws, never disables an upload the user did not disable)

A **missing, corrupt, or unknown/future** document falls back to the fresh-install defaults:

```json
{
  "control_schema_version": 1,
  "enabled": true,
  "mode": "allowlist",
  "siteRules": [{ "domain": "chatgpt.com", "enabled": true }, "…all built-ins…"]
}
```

- Unknown/future `control_schema_version` (or a non-object) ⇒ full defaults (on, `"allowlist"`,
  built-in AI rules).
- `mode: "disabled"` (a legacy document) ⇒ `{ "enabled": false, "mode": "all" }` with the
  sanitized rules kept.
- A valid `mode` (`"all"`/`"allowlist"`) ⇒ **the stored mode is kept** (`"all"` stays `"all"`),
  `enabled` is read as stored, and a missing/non-boolean `enabled` defaults to `true` (never
  block an upload on a half-written document).
- No usable `mode` but a boolean `enabled` ⇒ `{ "mode": "all", "enabled": <value> }`: the
  **pre-modes** behavior is historical all-sites, so it is never silently narrowed to allowlist.
- Anything else ⇒ defaults.
- Each rule's domain is normalized; invalid/duplicate rules are dropped.
- A stored `pausedUntil` is ignored: the pause was transient and is not migrated.
- Missing/corrupt storage or a storage error means "enabled": the extension must never block an
  upload because it could not read its own policy.

**Fresh default vs existing config.** Only a truly absent/non-object/unknown-version document
gets the new allowlist default. Every document that already carries a usable `mode` — or the
older `enabled`-only shape — keeps its stored behavior. A user who was on "all websites" is
never silently moved onto the allowlist by an upgrade.

Caps: at most 200 rules, at most 253 characters per normalized domain.

### Migration table (no row becomes more permissive)

| Stored document (previous checkpoints) | Migrated |
|---|---|
| `{ mode: "disabled", … }` | `{ enabled: false, mode: "all" }`, rules kept |
| `{ mode: "all", enabled: false, … }` (older "disable everywhere") | `{ enabled: false, mode: "all" }`, rules kept |
| `{ mode: "all", enabled: true, … }` | unchanged (`"all"` kept) |
| `{ mode: "allowlist", enabled: true, … }` | unchanged |
| `{ mode: "allowlist", enabled: false, … }` | `{ enabled: false, mode: "allowlist" }` (off, mode preserved) |
| `{ enabled: false }` only | `{ enabled: false, mode: "all" }` (historical all-sites) |
| `{ enabled: true }` only | `{ enabled: true, mode: "all" }` (historical all-sites) |
| absent / corrupt / unknown version | fresh default (on, `"allowlist"`, built-in AI rules) |

### Drop delivery and the fail-open guarantee

The optimized chip is only shown after a **confirmed handoff**. The extension never reports
"optimized" for a file the site did not receive.

- **Click (picker)** — the `change` event's own `<input type=file>` is the site's confirmed
  upload path; the optimized files are assigned there and the adapter's event sequence is
  replayed.
- **Drop** — a drop is only intercepted when the selected adapter can hand the file back
  confidently:
  - the adapter exposes a connected `<input type=file>`, in which case the drop uses the same
    input path as the picker; or
  - the adapter explicitly opted into the synthetic `dragenter`/`dragover`/`drop` sequence,
    which is set only for a site that was live-accepted with it (currently Claude).
  - Otherwise the drop is left **completely untouched** (no `preventDefault`, no
    `stopImmediatePropagation`) and the site handles the original. A generic page with no
    confirmed mechanism therefore fails open rather than faking success.
- **Post-interception failure** — if the optimized file cannot be handed off, the originals are
  re-presented through the same path, the localized **fallback** chip is shown, and the record
  (if any) describes the original bytes with `fallback` status. No optimized metric/history
  entry is written and the success chip never appears.

### Gemini drag/drop limitation

Gemini drag/drop **cannot be reliably optimized** by the extension. Gemini's drag pipeline is
trusted-event based and exposes no assignable file input on its drag path, so an extension can
only dispatch synthetic events that Gemini does not treat as a real attachment.

The accepted behavior is **fail-open**: on a Gemini drag/drop the original file attaches and the
extension claims no optimization. A drop is only intercepted with a confirmed mechanism, and
Gemini has none on its drag path. Gemini **click** upload remains optimized, because there the
file input exists and the picker path is confirmed. The capability matrix therefore records
Gemini `dragDrop: "passthrough"` (not `"unverified"`): the drag path was live-verified to attach
the original file, which is the accepted behavior. Its user-facing meaning — the original file is
uploaded unchanged and is not optimized — is what the Settings row tag says.

### Drop-handoff diagnostics

Every intercepted (and deliberately non-intercepted) drop emits a stable, allowlisted line at the
`delivery-handoff` stage. They are **informational**: open DevTools → **Console**; `console.info`
lines are visible at the default **Info** level (Verbose is not required, though enabling Verbose
also shows them). Only stable codes are logged — never a filename, path, URL, page content, or chat
data.

```
smaller stage=delivery-handoff code=<code>
```

| Code | Meaning |
|---|---|
| `input-not-found` | The adapter has an input lookup and found no connected `<input type=file>` when the drop arrived. |
| `input-detached` | An input was resolved but is no longer connected at delivery. |
| `input-assignment-failed` | Assigning `files` to the input threw / returned false. |
| `input-events-dispatched` | Files were assigned to a real input and its events were dispatched. |
| `handoff-confirmed` | A real input handoff was confirmed (follows `input-events-dispatched`). |
| `synthetic-declined` | The adapter has no synthetic opt-in, so no synthetic sequence was attempted. |
| `synthetic-dispatched` | A synthetic sequence was dispatched — explicitly **not** proof of acceptance. |
| `synthetic-failed` | The synthetic sequence threw while dispatching. |
| `fallback-original-used` | The optimized handoff failed and the originals were re-presented. |

Notes:

- A confirmed input handoff logs `input-events-dispatched` **and** `handoff-confirmed`. A synthetic
  dispatch logs only `synthetic-dispatched` and is **never** reported as confirmed.
- "Dispatched but not consumed" is not directly observable. The extension logs the dispatch
  (`input-events-dispatched` / `handoff-confirmed`), and a site that silently ignores the files is
  inferred from the user-visible symptom, not detected.
- When a drop is left to the site (for example a Gemini drag), `input-not-found` and
  `synthetic-declined` are both emitted when the adapter has an input lookup; an adapter with no
  lookup at all (the generic adapter) logs only `synthetic-declined`.

## 4. Broad website scope

The extension runs on and may evaluate **any normal web page** (`http`/`https`). Broad access is
what lets a **custom** site be added and evaluated **without a new per-site permission prompt**.
It is not the same as "optimize everywhere": the default mode is the built-in AI allowlist, so on
a fresh install only the built-in AI websites are optimized even though the extension is present
on any page. Users can switch to "all websites" mode (or enable specific custom sites) at any
time.

Website access is needed to detect the images/videos you pick, drop, or paste **before** they
upload so they can be optimized locally. The extension does not read page text, chat content,
cookies, or your browsing history.

Rules therefore take effect on any HTTP/HTTPS site, not only the built-in AI websites.

## 5. Website Rules list

There is **one** rules list, with no separate preset area.

- **Built-in rows** — fixed, always present, and actionable. The names and domains are proper
  nouns and are never translated:

  | Name | Domain |
  |---|---|
  | ChatGPT | `chatgpt.com` |
  | Claude | `claude.ai` |
  | Gemini | `gemini.google.com` |
  | DeepSeek | `chat.deepseek.com` |
  | Perplexity | `perplexity.ai` |
  | Grok | `grok.com` |
  | Microsoft Copilot | `copilot.microsoft.com` |
  | Poe | `poe.com` |
  | Mistral Le Chat | `chat.mistral.ai` |

  On a fresh install every built-in domain is present and enabled, so the default allowlist is
  meaningful immediately.

  **Capability status.** Each built-in site carries a capability matrix of four flows
  (`clickUpload`, `dragDrop`, `image`, `video`), each with one status from an explicit
  vocabulary:

  | Status | Meaning |
  |---|---|
  | `supported` | The flow is verified to work: the upload is optimized. |
  | `passthrough` | The flow is verified to **upload the original file unchanged** (not optimized). This is the accepted behavior, not a failure. |
  | `pending` | An adapter/implementation exists but real-site verification has not passed yet; never presented as supported. |
  | `unverified` | Not tested; must never be presented as supported. |

  The Settings page derives one localized tag per row — a `passthrough` drag shows the
  limitation text, a `pending` drag shows the verification-pending text (an implementation
  exists but is not yet verified), an unverified site shows **"Not verified"**, and a fully
  supported site shows no tag. None of these tags is a permission limitation: the broad site
  permission above applies to every website, and every row is fully selectable.

  | Name | `clickUpload` | `dragDrop` | `image` | `video` |
  |---|---|---|---|---|
  | ChatGPT | `supported` | `supported` | `supported` | `unverified` |
  | Claude | `supported` | `supported` | `supported` | `unverified` |
  | Gemini | `supported` | `passthrough` | `supported` | `unverified` |
  | DeepSeek | `supported` | `supported` | `supported` | `unverified` |
  | Perplexity | `unverified` | `unverified` | `unverified` | `unverified` |
  | Grok | `unverified` | `unverified` | `unverified` | `unverified` |
  | Microsoft Copilot | `unverified` | `unverified` | `unverified` | `unverified` |
  | Poe | `unverified` | `unverified` | `unverified` | `unverified` |
  | Mistral Le Chat | `unverified` | `unverified` | `unverified` | `unverified` |

  **DeepSeek** has a dedicated adapter. Its click upload and drag/drop are both **supported**,
  verified on a real logged-in Chrome session: the optimized file attaches through the site's
  real `<input type=file>` (the same confirmed input mechanism as ChatGPT, with `input`/`change`
  replayed). A completion-only, empty-transfer `dragleave`/`drop`/`dragend` lifecycle closes the
  site's drag overlay (the same ChatGPT-style cleanup). The adapter deliberately does **not** opt
  into the synthetic drop sequence and never selects a code-only upload input, so a drop with no
  usable image input is left to the site (fail open).

- **Custom rows** — domains you add.

Each row has a **selection checkbox** and shows its **effective state** (on/off) under the
current switch + mode + rule. The three bulk actions own mutation:

| Action | Effect |
|---|---|
| **Enable selected** | Adds/updates an explicit `{ enabled: true }` rule for each selected domain. |
| **Disable selected** | Adds/updates an explicit `{ enabled: false }` rule for each selected domain. |
| **Delete selected** | Removes the explicit rule for each selected domain. |

The bulk actions only add/update/remove explicit `siteRules` entries; they **never change the
global switch or `mode`**, so selecting a row can never silently enable or disable other
websites. Per-row toggle/delete buttons do not exist any more.

## 6. Domain matching

Domains are normalized to a bare lower-case hostname: scheme, userinfo, port, path, query,
fragment, surrounding whitespace, and trailing dots are stripped. A wildcard is not supported;
an empty or structurally invalid domain is rejected.

A rule matches the page host exactly, or as a parent domain:

```
rule: example.com
host: example.com        -> match
host: www.example.com    -> match
host: notexample.com     -> no match
host: example.com.evil   -> no match
```

When rules overlap, the most specific (longest) matching domain wins.

## 7. Supported file types

The file-type allowlist is extension-based (case-insensitive):

| Kind | Extensions |
|---|---|
| Image | `jpg`, `jpeg`, `png`, `webp`, `heic`, `heif` |
| Video | `mp4`, `mov`, `m4v` |

- MIME type is **not** used to admit a file; an extension-less or mislabelled file is left to
  the site.
- Anything else (`pdf`, `doc`, `docx`, `zip`, `txt`, `webm`, unknown, …) is never intercepted:
  no `preventDefault`, no native call, no chip, and no metrics/history entry.
- In a mixed batch, supported files are optimized and unsupported files are re-presented to the
  site as the original `File`, in order.
- `webm` is not supported by Core and is skipped silently.
- `tif`/`tiff` are Core CLI-supported but excluded from this browser allowlist for now. This is
  a deliberate, documented boundary.

## 8. Export format

Export produces pretty-printed JSON in the Settings page (and can be downloaded as
`smaller-please-config.json`). The shape is stable, human-readable, and convenient for an AI
assistant to read and reason about:

```json
{
  "control_schema_version": 1,
  "enabled": true,
  "mode": "allowlist",
  "siteRules": [{ "domain": "chatgpt.com", "enabled": true }]
}
```

`enabled` is written **as stored** (it is the global switch, not a derived field), and there is
no `pausedUntil` field any more.

## 9. Import rules and compatibility

Import accepts the exported document and a small set of load-compatible aliases. It is strict
where sanitize-on-read is lenient, and **no import path is more permissive than its input**:

- A missing `control_schema_version` defaults to `1`; any other value is rejected.
- `enabled`, when present, must be a boolean.
- `mode` present:
  - `"disabled"` ⇒ `{ enabled: false, mode: "all" }` (rules kept);
  - `"all"`/`"allowlist"` ⇒ that mode, with `enabled` as given or `true` when absent;
  - anything else ⇒ rejected (`unsupported-mode`).
- `mode` absent ⇒ `"all"`, with `enabled` as given or `true` when absent.
- The canonical rules array is `siteRules`; a legacy `rules` array is accepted as an alias.
  When both are present, **canonical `siteRules` wins**.
- Each rule is `{ domain: string, enabled: boolean }` or the alias
  `{ domain: string, action: "enable" | "disable" }`. The alias is per item, so a mixed document
  still loads. An unknown action or a non-boolean `enabled` is rejected.
- A document with neither `siteRules` nor `rules` has **no** rules (so a minimal `{ "mode":
  "all" }` or `{ "rules": [] }` loads).
- Malformed rules, invalid/empty domains, duplicate domains, and more than 200 rules are
  rejected.
- `pausedUntil` is never imported.
- Import replaces the whole `enabled`/`mode`/`siteRules` set after a confirmation prompt.

### Compatibility table

| Document | Accepted on import | Result |
|---|---|---|
| v1 canonical: `{ control_schema_version, enabled, mode, siteRules: [{ domain, enabled }] }` | yes | loaded as-is (missing version defaults to `1`) |
| Alias rules: `{ mode, rules: [{ domain, action: "enable"\|"disable" }] }` | yes | `action` mapped to `enabled` |
| Mixed items in one array (canonical and alias rows) | yes | each item parsed independently |
| Both `siteRules` and `rules` present | yes | canonical `siteRules` wins |
| `enabled`-only legacy: `{ enabled: boolean }` | yes | `mode` becomes `"all"`; the switch keeps the value |
| Legacy three-mode: `{ mode: "disabled", … }` | yes | `{ enabled: false, mode: "all" }`, rules kept |
| Previous-checkpoint `{ mode: "all", enabled: false, … }` | yes | off switch, `mode: "all"`, rules kept |
| Minimal `{ "mode": "all" }` or `{ "rules": [] }` | yes | no rules |
| Unknown/future `control_schema_version` | no | `unsupported-version` |
| Unknown `mode` value | no | `unsupported-mode` |
| Unknown rule `action`, malformed rule, duplicate domain, over cap | no | `invalid-rules` / `too-many-rules` |
| Invalid/non-normalizable domain | no | `invalid-domain` |
| Non-boolean top-level `enabled` | no | `invalid-document` |

Each rejection maps to a stable code that the UI localizes; the raw JSON is never rendered:

| Code | UI meaning |
|---|---|
| `invalid-json` | not valid JSON |
| `invalid-document` | not a configuration document |
| `unsupported-version` | unsupported configuration version |
| `unsupported-mode` | unsupported configuration mode |
| `invalid-rules` | malformed website rules |
| `invalid-domain` | invalid website domain |
| `too-many-rules` | too many website rules |

## 10. Settings page and popup

The Settings page is registered as the extension's options page (opened in a tab) and opened
from the popup. It holds every advanced control, in this order:

- **General** — the one global switch, a status line for the current policy, and the language
  selector.
- **Website Control** — a radio group with exactly two options (Optimize on all websites / Only
  selected websites). A fresh install checks **Only selected websites** (the built-in AI
  allowlist); the note explains the selected mode and why broad website access is needed to add
  custom sites without extra prompts.
- **Website Rules** — the built-in rows and the custom rows (selection checkbox + effective
  state), the add-website input, and the three bulk actions.
- **Compression** — the Core-owned image/video minimum sizes and minimum savings, read and
  written through `config_get`/`config_set` (with an "unavailable" notice when Core is down).
- **Storage** — the Core-owned store location/usage, the apply-location action, and the
  confirmed clean action.
- **History** — the browser-local history limit (with the empty/garbage guard), clear history,
  and reset local metrics.
- **Advanced** — export configuration (textarea + download) and import (paste or choose a JSON
  file, with a confirmation step).
- **About** — the extension version, the Core version/status from a handshake (localized; no raw
  native text), and **Open install instructions**.

The popup is deliberately high-frequency-only. It shows the Core status, the same single global
switch and its policy status line, the current site's **concise** effective state
(`{host} · {Enabled|Disabled}`), compression statistics, the recent-activity list, and
**Open Settings**. A reason for the current site's state (if any) lives behind the collapsed
**Why?** diagnostics disclosure in the popup rather than in the body text. The switch toggles
`enabled` only and never changes `mode` or a rule.

## 11. Non-goals (explicit)

- **No AI auto-editing.** An AI assistant can read/analyze the exported JSON, but Smaller,
  Please does not let an AI or any process edit the configuration for you.
- **No CLI / Native Host management of this config.** The control document is browser-local;
  `smaller config` does not read or write it, and the Native Messaging protocol has no control
  operation.
- **No threshold duplication.** Optimization thresholds remain Core-owned.
- **"Not verified" and `passthrough` describe tested support, not a missing permission.** Every
  built-in row is selectable and the broad site permission already covers it. `unverified` means a
  flow has not been live-accepted yet; `passthrough` means a flow is verified to upload the
  original file unchanged (an accepted limitation, never a permission limitation).
- **No per-site thresholds, schedules, or wildcard rules** in this phase.
