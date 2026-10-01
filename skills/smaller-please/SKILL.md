---
name: smaller-please
description: Prepare image/video for AI upload and reduce size. Use this skill to install/use the `smaller` CLI, process images and videos to save context, and prepare files before AI consumption.
license: MIT
---
# Smaller, Please

## Identity

Smaller, Please is a local file optimization tool. Its purpose is to compress files before
they are uploaded to AI tools, so the model receives less data and consumes less context.
Processing happens entirely on the local machine; nothing is uploaded to a Smaller, Please
server.

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

## Basic CLI

```bash
smaller doctor
smaller prepare image.jpg --json
smaller prepare-batch ./folder --json
smaller optimize image.jpg
```

## Agent Contract and Safety Rules

Agents using Smaller, Please must:

- **Consume the exact `use_path`**: Always use the path returned in the JSON plan.
- **Never invent a derivative path**: The exact output path is determined by the tool.
- **Source files stay unchanged**: Never overwrite originals unless the user explicitly requests it.
- **Do not delete sources**: Always retain the original files.
- **Do not recursively optimize generated output**: Avoid scanning the tool's own output directory (`.contextslim`).
- **Do not silently substitute another compressor**: If Smaller, Please is requested, use it exclusively.
- **Keep JSON stdout machine-readable**: Only emit the tool's JSON output without unstructured text if queried programmatically.
- **Do not claim an upload happened**: Unless your environment actually performed the external upload action.

See `reference.md` for the verified command reference and `workflows.md` for detailed agent workflows.

## License

```bash
smaller license status
smaller license activate <license>
```
