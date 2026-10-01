# Smaller, Please CLI Guide

This is the technical guide for the `smaller` CLI command, designed for humans and AI agent authors. 

## Command Reference

The single optimization command is `smaller optimize <INPUT>`. The source file is never modified or overwritten.

```bash
smaller optimize [OPTIONS] <INPUT>
```
**Input**: An image, a video, or a directory (directories are recursively optimized).

**Options:**
- `--mode <smart|maximum>` (default: `smart`): `smart` uses the default safety pipeline. `maximum` compresses more aggressively but requires the `compression.maximum` capability (Pro).
- `--metadata <remove-all>`: Strips all removable metadata. Requires the `metadata.control` capability (Pro). Without this flag, metadata handling depends on the underlying backend.
- `--max-edge` (default: 2048): The maximum length in pixels for the longest edge of images/videos.
- `--quality`: Adjust the quality setting.
- `--crf`: Adjust the CRF for video encoding.
- `--min-savings`: The minimum percentage size reduction required to keep the optimized file.
- `--output`: Define a specific output directory. Default is `.contextslim/` next to the source.
- `--force`: Bypass the minimum source-size gate (a CLI-only opt-in for `optimize`).
- `--dry-run`: Do not write any files; simulate the process.
- `--json`: Output machine-readable JSON format instead of human-readable text.
- `--image-min-size-mb`: The minimum file size in MiB for an image to be processed (default 2 MiB, `0` always attempts).
- `--video-min-size-mb`: The minimum file size in MiB for a video to be processed (default 10 MiB, `0` always attempts).
- `--min-size-mb`: Global fallback for minimum size in MiB.

## AI Agent Commands

The `prepare` commands emit a frozen JSON plan with the `use_path` field and a `status` field. The output is machine-readable on stdout, with diagnostics printed to stderr. 

```bash
smaller prepare [OPTIONS] <PATH>
```
Prepares one image or video for agent input. The returned JSON document contains the exact `use_path` you should consume.

```bash
smaller prepare-batch [OPTIONS] <PATHS>...
```
Prepares several files or directories, returning one JSON document. Each item is handled independently.

**Options for `prepare` and `prepare-batch`:**
- `--max-edge`, `--quality`, `--crf`, `--min-savings`, `--output`, `--json`, `--image-min-size-mb`, `--video-min-size-mb`, `--min-size-mb`.

### JSON Contract
- `status` will be one of:
  - `optimized`: `use_path` points to the smaller derivative. Use it.
  - `skipped`: Source is below the minimum size threshold. `use_path` points to the original source. Use it.
  - `fallback`: Optimization didn't improve the file. `use_path` points to the original source. Use it.
  - `error`: Source is unusable. 
- Always consume the exact `use_path` provided.
- Never invent a derivative path.
- Source files must remain preserved.

## System Health & Setup

- `smaller doctor [--json]`: Shows backend availability and checks installation health.
- `smaller setup [--json] [--yes]`: Sets up Core configuration, managed store, media engine, and native host. Does not install Homebrew.
- `smaller repair [--dry-run] [--yes] [--json]`: Repairs allowlisted installation problems.

## Configuration & Storage

- `smaller config show|get|set|reset`: Read or update the persisted Core configuration.
- `smaller clean [store|cache|inbox|all] [--dry-run] [--yes] [--json]`: Report or remove cached media managed by Smaller, Please.

## Components

- `smaller extension install|status|path|open|uninstall`: Manage the staged Chrome extension bundle.
- `smaller native install|status|uninstall|host`: Manage the Chrome Native Messaging host.
- `smaller media status|install|path|uninstall`: Install or manage the optional LGPL Media Pack.

## License Management

- `smaller license activate <KEY> [--json]`: Verify and store a license key locally (no network). 
- `smaller license status [--json]`: Show the local license status.
- `smaller license remove [--json]`: Delete the local license file and return to Free.

## Uninstall

- `smaller uninstall [--incomplete] [--dry-run] [--yes] [--json] [--remove-settings] [--remove-cache]`: Removes program/integration files, but keeps config, store/cache, browser-local data, Homebrew, and user media by default.
