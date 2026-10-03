# Smaller, Please — workflows

## Trigger boundary

Explicit intent or explicit user policy triggers Smaller, Please; mere media presence does not. Do not intercept or optimize media just because it exists or is being analyzed. AI preparation must use `smaller prepare` or `smaller prepare-batch`, not `smaller optimize`.

## AI upload workflow

When a user asks to prepare a file for upload to an AI tool:

1. Resolve the input once using the ordered contract below.
2. Run `smaller prepare "<path>" --json` locally.
3. Parse the JSON and consume the exact `use_path`.
4. Stop; do not add discovery or verification probes on the ordinary success path.
5. Do **not** claim an upload happened unless the environment actually performed it.

Example request: "Prepare this image for ChatGPT upload."

```bash
smaller prepare ./photo.png --json
```

Report the resulting `use_path` to the user as the file to attach. Uploading stays the user's
action.

### Input resolution contract

Resolve an attachment in this order:

1. Use the authoritative local filesystem path when the environment provides one.
2. Otherwise, use the environment's standard attachment/file API or materialization mechanism
   once, then pass the resulting path.
3. If only bytes/content are exposed, materialize them once through the environment's normal
   temp or workspace mechanism. Preserve the real media extension when it is known, then pass the
   materialized path.
4. If neither a path nor a content/materialization mechanism is available, ask the user for a
   local file path.

Do not discover attachments by crawling all of HOME; guessing names in unrelated Downloads or
Desktop folders; inspecting app, browser, or agent caches/private databases; querying SQLite or
tool-output stores; or searching broad temp directories, shell history, logs, or unrelated project
directories. These are private implementation details, not attachment APIs.

The normal path is exactly: authoritative path → `smaller prepare "<path>" --json` → parse JSON →
consume exact `use_path` → done. Do not run `which smaller`, inspect installation directories,
repeat Skill discovery, call `--help`, or probe with `ls`, `file`, or `sips` unless `prepare`
returned an error or fallback that actually requires diagnosis.

## Agent prepare workflow

`smaller prepare --json` writes exactly one machine-readable JSON document to stdout
(diagnostics go to stderr). The plan is versioned and frozen; treat it as a stable contract.

### Contract and Rules

- **Consume the exact `use_path`**: Read the `use_path` field and use that **exact** file. Never guess or construct the derivative path yourself.
- **Source files stay unchanged**: Source files are preserved. Never delete them.
- **Do not recursively optimize generated output**.
- **Do not silently substitute another compressor** when Smaller, Please is requested.
- **Keep JSON stdout machine-readable**.
- Treat `source_unchanged`, `original_bytes`, `optimized_bytes`, `saved_percent`, and `use_path`
  as authoritative. Do not re-check them with filesystem, media, or hashing probes on ordinary
  success.

`status` is one of `optimized`, `skipped`, `fallback`, or `error`:

- `optimized` — `use_path` is the smaller derivative; use it.
- `skipped` — the source was below the size gate; `use_path` is the original source.
- `fallback` — optimization could not improve the file; `use_path` is the original source.
- `error` — the source itself is unusable; `use_path` is `null`; report it.

On `skipped` and `fallback`, `use_path` refers to the source. On `error`, there is no safe path.
Never fabricate an output path, and never substitute another compressor.

### Success fast path — no post-success verification

A valid JSON plan with `status` of `optimized`, `skipped`, or `fallback` ends the job. Do not run
post-success verification.

- Do not run `ls`, `file`, `sips`, `shasum`/hashing, `which`, `command -v`, `env`,
  `smaller --version`, `smaller config ...`, or any filesystem/config inspection merely to verify
  the prepare result.
- Treat Core JSON as authoritative for `source_unchanged`, `output`, `use_path`, `original_bytes`,
  `optimized_bytes`, `saved_percent`, `cache_hit`, and `backend`.
- The normal success path is exactly: obtain the authoritative input path →
  `smaller prepare "<path>" --json` → parse JSON → consume/report the exact `use_path` → done.
- Diagnostics are allowed only after: invalid JSON; a nonzero/`error` result; an explicit user
  request to diagnose the installation/environment; or a result that is internally inconsistent.

`smaller prepare-batch --json` handles each item in a batch independently; one `error` item
does not invalidate the others.

### Image example

```bash
smaller prepare ./diagram.png --json
```

Read `use_path` from the JSON and attach that file.

### Video example

```bash
smaller prepare ./clip.mov --json
```

Read `use_path` from the JSON and attach that file.
