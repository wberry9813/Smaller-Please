# Smaller, Please

**Locally prepare smaller AI-ready files before upload while preserving originals.**

[Read in Simplified Chinese (简体中文)](README.zh-CN.md)

## Install

Homebrew is the only supported installation method (macOS 12+ on Apple Silicon (arm64) only).

```bash
brew tap wberry9813/smaller-please
brew install smaller-please
smaller setup
smaller doctor
```

Then load the browser extension manually (see below).

## What it does

- **Image optimization:** HEIC, JPEG, and TIFF are optimized into smaller JPEG or PNG copies.
- **Video optimization:** MP4, MOV, and M4V are compressed locally using H.264 + AAC.
- **Browser upload optimization:** Intercepts and shrinks uploads on supported AI websites.
- **AI-agent preparation:** CLI tools output machine-readable JSON for agents to use optimized files.
- **Local processing:** Everything runs on your Mac. No media is sent to our servers.
- **Source preservation:** Original files are never modified or deleted by default.
- **Smart and Maximum compression:** 'Smart' mode for safe defaults, 'Maximum' for aggressive savings (requires Pro).
- **Metadata control:** Explicitly remove all metadata with `--metadata remove-all` (requires Pro).

## Browser extension

After installing with Homebrew, you must manually load the extension:

1. Run `smaller setup` to prepare the extension files.
2. Run `smaller extension path` to reveal the exact folder to use (`~/Applications/Smaller Please Extension`).
3. Open `chrome://extensions` in Google Chrome.
4. Enable **Developer mode** (top-right).
5. Click **Load unpacked** and select the folder from step 2.

**Supported Sites:**
The extension runs by default on exactly three built-in AI websites:
- **ChatGPT** (`chatgpt.com`): Click and drag-and-drop supported.
- **Gemini** (`gemini.google.com`): Click supported. Drag-and-drop is **passthrough** (the original file attaches unchanged and is not optimized).
- **DeepSeek** (`chat.deepseek.com`): Click and drag-and-drop supported.

## CLI

The `smaller` command helps you optimize media and check system health.

```bash
smaller optimize ./photo.jpg
smaller optimize ./clip.mov
smaller prepare ./photo.jpg --json
smaller prepare-batch ./input --json
smaller doctor
```

- `optimize`: The single command for optimizing an image, video, or directory.
- `prepare` / `prepare-batch`: Emits a frozen JSON plan containing `use_path`. `use_path` is the exact file you should upload or use.

## Use with AI agents

Smaller, Please provides a first-class public Agent Skill for use with OpenCode and Codex (verified discovery) and Claude Code (compatible via the shared Agent Skills standard).
AI agents must install the `smaller` CLI and follow strict guidelines:
- Prefer `smaller prepare` or `smaller prepare-batch --json` to get the optimization plan.
- Always consume the exact `use_path` returned in the JSON plan.
- Never invent a derivative output path.
- Source files are preserved; never modify or delete them.
- Never silently substitute another compressor.
- Uploads are subject to the environment's actual capability (do not claim an upload happened unless the environment actually performed it).

Read the Agent Skill instructions:
- [Skill Definition](skills/smaller-please/SKILL.md)
- [Workflows](skills/smaller-please/workflows.md)
- [Command Reference](skills/smaller-please/reference.md)
- [Agent Skill Guide](docs/AGENT_SKILL.md)

## Free and Pro

- **Free:** Includes `smart` mode optimization, browser optimization, and default metadata behavior.
- **Pro:** Adds `maximum` compression mode and metadata removal (`remove-all`) via a locally verified licence key. Purchasing is not yet generally available.

## Privacy

Your media processing is strictly local. Smaller, Please does not upload your images or videos to any media-processing server. Read the [Canonical Privacy Policy](PRIVACY.md).

## Update

Update through Homebrew and then refresh your local setup:

```bash
brew update
brew upgrade smaller-please
smaller setup
smaller doctor
```
After upgrading, go to `chrome://extensions` and click the refresh/reload icon on the Smaller, Please extension card.

## Uninstall

```bash
brew uninstall smaller-please
smaller uninstall
```
See [Uninstall Guide](docs/uninstall.md) for details on what is kept by default.

## Troubleshooting

See the troubleshooting guides for the
[browser extension](docs/troubleshooting/browser-extension.md),
[native host](docs/troubleshooting/native-host.md),
[core](docs/troubleshooting/core.md),
[storage](docs/troubleshooting/storage.md), and
[media engine](docs/troubleshooting/media-engine.md).

## License

MIT. See [`LICENSE`](LICENSE) and [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md).

## Website / Releases

- **Website:** [https://smaller-please.inchmirror.studio](https://smaller-please.inchmirror.studio)
- **Releases:** [https://github.com/wberry9813/Smaller-Please/releases](https://github.com/wberry9813/Smaller-Please/releases)
