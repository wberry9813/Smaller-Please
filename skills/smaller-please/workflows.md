# Smaller, Please — workflows

## AI upload workflow

When a user asks to prepare a file for upload to an AI tool:

1. Inspect the file (path, type, size).
2. Run Smaller, Please locally on it.
3. Return the optimized file path to the user.
4. Do **not** claim an upload happened unless the environment actually performed it.

Example request: "Prepare this image for ChatGPT upload."

```bash
smaller prepare ./photo.png --json
```

Report the resulting `use_path` to the user as the file to attach. Uploading stays the user's
action.

## Agent prepare workflow

`smaller prepare --json` writes exactly one machine-readable JSON document to stdout
(diagnostics go to stderr). The plan is versioned and frozen; treat it as a stable contract.

### Contract and Rules

- **Consume the exact `use_path`**: Read the `use_path` field and use that **exact** file. Never guess or construct the derivative path yourself.
- **Source files stay unchanged**: Source files are preserved. Never delete them.
- **Do not recursively optimize generated output**.
- **Do not silently substitute another compressor** when Smaller, Please is requested.
- **Keep JSON stdout machine-readable**.

`status` is one of `optimized`, `skipped`, `fallback`, or `error`:

- `optimized` — `use_path` is the smaller derivative; use it.
- `skipped` — the source was below the size gate; `use_path` is the original source.
- `fallback` — optimization could not improve the file; `use_path` is the original source.
- `error` — the source itself is unusable; report it.

On `skipped`, `fallback`, and `error`, `use_path` refers to the source — use it rather than
fabricating an output path.

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
