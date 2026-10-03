---
name: smaller-please
description: Compress, shrink, or prepare local images/videos for AI upload or to reduce size, on explicit user request. For AI preparation agents use `smaller prepare` / `smaller prepare-batch`, never `smaller optimize`. Use when the user explicitly asks for Smaller, Please, explicitly asks to compress/reduce/shrink/prepare a media file, or a user-owned project/workspace rule requires it. Do NOT trigger merely because an image or video exists or is being analyzed or uploaded.
license: MIT
---
# Smaller, Please

## Identity

Smaller, Please is a local file optimization tool. Its purpose is to compress files before
they are uploaded to AI tools, so the model receives less data and consumes less context.
Processing happens entirely on the local machine; nothing is uploaded to a Smaller, Please
server.

## When to Use (Trigger Policy)

- Explicit intent or explicit user policy triggers Smaller, Please. Mere media presence does not.
- Trigger cases: (1) explicit Smaller, Please / smaller request; (2) explicit capability intent: compress this image/video, make this smaller, reduce upload size, shrink before giving it to AI, prepare this image/video for AI (brand mention not required when intent is explicit); (3) the user's own project/workspace/agent rule explicitly requires Smaller, Please, and that rule is USER-OWNED policy that Smaller never installs or mutates.
- Non-triggers: do not trigger because an image or video exists, a local path is referenced, media is about to be inspected, analyzed, or uploaded, the file looks large, or compression could save bandwidth/context/tokens.
- Non-trigger examples: Analyze this image. What is in this picture? Summarize this video. Look at /path/to/video.mp4. Upload this image.
- No proactive media interception.
- For AI preparation the Skill uses `smaller prepare` / `smaller prepare-batch`, never `smaller optimize`; optimize is the human/general explicit optimization surface.
- The Browser Extension is a separate, explicitly-enabled automatic upload surface; installing the Skill does not authorize automatic interception.

## Surfaces

1. **CLI** — the `smaller` command, used by humans and by AI agents to prepare files for
   upload.
2. **Browser extension** — automatically optimizes supported uploads on supported AI
   websites.

## Installation

Install the macOS release with Homebrew:

```bash
brew tap wberry9813/smaller-please
brew install smaller-please
```

Then finish setup and add the browser extension (the extension is bundled with the install):

```bash
smaller setup           # config + managed store + Media Engine + Native Host + extension staging
smaller extension path  # prints the folder to select in Chrome
```

In Chrome: open `chrome://extensions`, enable **Developer mode**, click **Load unpacked**, and
select the folder printed by `smaller extension path` (a stable path, not a versioned install
directory).

Verify the installation:

```bash
smaller doctor
```

### Agent Skill Installation

The bundled Agent Skill is the SOURCE OF TRUTH. Run `smaller get skills` to view the local Skill path and installation instruction.

1. **PREFERRED: Symbolic Link**
   Create a symbolic link from the agent ecosystem's skill directory (e.g., `~/.agents/skills/smaller-please`) to the stable bundled Skill path (e.g., `/opt/homebrew/share/smaller-please/skills/smaller-please`).
   - This ensures `brew upgrade smaller-please` updates the Skill automatically via the stable symlink.
   - **NEVER** symlink to a versioned Cellar path.
   - If a previous copied snapshot exists at the destination, replace it with the symlink.
2. **FALLBACK ONLY: Copy**
   If the ecosystem cannot use symlinks, copy the Skill files. A copied Skill becomes a stale snapshot and must be manually re-installed after a future upgrade.

## Basic CLI

Agents preparing media for AI must use `smaller prepare` / `smaller prepare-batch`, never `smaller optimize`. `smaller optimize` is the human/general explicit optimization surface.

```bash
smaller doctor
smaller prepare image.jpg --json
smaller prepare-batch ./folder --json
smaller optimize image.jpg
```

## Agent Contract and Safety Rules

> **No post-success verification.** Once `smaller prepare "<path>" --json` returns valid JSON
> with `status` of `optimized`, `skipped`, or `fallback`, the workflow is complete: parse the
> JSON and consume/report the exact `use_path`. Do not run any of these merely to confirm the
> result: `ls`, `file`, `sips`, `shasum`/hashing, `which`, `command -v`, `env`,
> `smaller --version`, `smaller config ...`, or any filesystem/config inspection. Core JSON is
> authoritative for `source_unchanged`, `output`, `use_path`, `original_bytes`, `optimized_bytes`,
> `saved_percent`, `cache_hit`, and `backend`. Diagnostics are allowed only after invalid JSON, a
> nonzero/`error` result, an explicit user request to diagnose the installation/environment, or an
> internally inconsistent result.

Agents using Smaller, Please must:

- **Resolve input without searching private state**: Use the environment's authoritative local
  filesystem path first. Otherwise, use its standard attachment/file API once to obtain or
  materialize the file. If only bytes/content are exposed, materialize once through the normal
  temp/workspace mechanism and preserve the real media extension when known. If neither a path nor
  a content mechanism exists, ask the user for a local file path.
- **Use the fast path**: Obtain the authoritative path, run
  `smaller prepare "<path>" --json`, parse the JSON, consume the exact `use_path`, and stop.
- **Consume the exact `use_path`**: Always use the path returned in the JSON plan.
- **Never invent a derivative path**: The exact output path is determined by the tool.
- **Config is not output**: `~/Library/Application Support/SmallerPlease` is config, not
  derivative/output storage. A `prepare` derivative lives under the managed store
  (`~/.contextslim-bridge/store/derivatives/`); never construct or guess it — consume the exact
  `use_path`.
- **Source files stay unchanged**: Never overwrite originals unless the user explicitly requests it.
- **Do not delete sources**: Always retain the original files.
- **Do not crawl for attachments**: Never search all of HOME, guess filenames in unrelated
  Downloads/Desktop folders, or inspect app/browser/agent caches, private databases, SQLite or
  tool-output stores, broad temp directories, shell history, logs, or unrelated projects.
- **Do not recursively optimize generated output**: Avoid scanning the tool's own output.
- **Do not silently substitute another compressor**: If Smaller, Please is requested, use it exclusively.
- **Keep JSON stdout machine-readable**: Only emit the tool's JSON output without unstructured text if queried programmatically.
- **Do not claim an upload happened**: Unless your environment actually performed the external upload action.
- **Trust the plan on ordinary success**: Treat `source_unchanged`, `original_bytes`,
  `optimized_bytes`, `saved_percent`, and `use_path` as authoritative; do not re-run
  `ls`, `file`, `sips`, or hashing merely to confirm them (see the success fast path above).

See `reference.md` for the verified command reference and `workflows.md` for detailed agent workflows.

## License

```bash
smaller license status
smaller license activate <license>
```
