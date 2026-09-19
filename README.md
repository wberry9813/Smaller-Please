# Smaller Please

**Make uploads smaller.**

Smaller Please compresses images and videos on your Mac before they are uploaded to AI apps
such as ChatGPT and Claude. Everything happens locally — no cloud processing, no account, no
telemetry.

[Download the beta](https://github.com/wberry9813/Smaller-Please/releases/tag/v0.1.0-beta.3) ·
[Install guide](docs/DOWNLOAD.md) · [Privacy](docs/PRIVACY.md) · [中文](README.zh-CN.md)

## What it is

Large screenshots, photos, and exported UI images can waste upload and proxy traffic when an AI
tool only needs a visually faithful working copy. Smaller Please creates a smaller, AI-ready
copy of each image or video and reports the bytes saved — without touching your original file.

It works from the command line, and it integrates with Chrome so the images and videos you pick,
drop, or paste into any website (for example ChatGPT or Claude) are optimized before they are
uploaded.

## Features

- **Local image compression** — HEIC/JPEG/TIFF → JPEG and PNG handling, with a never-grow guard
  and a minimum-savings threshold.
- **Local video compression** — MP4/MOV/M4V → MP4 (H.264 + AAC), with the same never-grow guard.
- **Chrome integration** — optimizes images and videos you pick, drop, or paste into a website
  (for example ChatGPT or Claude).
- **One switch, two website modes, and per-site rules** — by default Smaller runs only on a
  built-in AI allowlist (ChatGPT, Claude, Gemini, DeepSeek, Perplexity, Grok, Microsoft Copilot,
  Poe, Mistral Le Chat); switch to all-sites mode at any time and manage one unified rules list
  for the built-in sites and any website you add.
- **Human-readable, exportable config** — export your rules as plain JSON you (or an AI
  assistant) can read and keep.
- **Never grows your files** — if the optimized copy is not smaller, it is discarded and the
  original is used.
- **No cloud processing** — files stay on your Mac.
- **macOS installer** — a user-level install with no password. The Beta.3 DMG is signed with a
  Developer ID and notarized/stapled by Apple, so Gatekeeper accepts it.

Sources are never modified or overwritten.

## Control what gets optimized

Open the extension popup to flip the single **Smaller** on/off switch and see the effective
state for the current site. Open **Settings** for the full controls:

- Toggle optimization on/off everywhere with the **Smaller** switch — your mode and website
  rules are kept.
- Choose a **website mode**: optimize on all websites, or only on selected websites.
- Use the **Website Rules** list — the **Built-in** AI websites (ChatGPT, Claude, Gemini,
  DeepSeek, Perplexity, Grok, Microsoft Copilot, Poe, Mistral Le Chat) and your **Custom**
  websites — or add any website.
- **Export** your configuration as readable JSON, and **import** it back (or paste one in).

Smaller has broad access to websites so you can add a custom site without a new permission
prompt each time. **By default it runs only on the built-in AI allowlist** ("Only selected
websites"); you can switch to "Optimize on all websites" at any time. The switch always wins,
then a website rule, then the mode, so a website that is not listed is not optimized in
allowlist mode. It only looks at the file you pick, drop, or paste.

Rows for sites whose upload path has not yet been live-accepted are marked **"Not verified"** —
that reflects Smaller's tested support for the site, and is **never** a permission limitation. A
row can instead carry a **`passthrough`** capability: that flow is verified to upload the original
file unchanged, an accepted limitation rather than a failure. **On Gemini, click uploads are
optimized, while drag/drop uploads the original file unchanged and is not optimized.** The
fresh-install default is the built-in AI allowlist, but an existing configuration is never
silently changed. A dropped file is only reported as optimized after it is handed to the site's
own upload path; if that handoff cannot be confirmed, the original file is uploaded instead. See
[`docs/EXTENSION_SETTINGS.md`](docs/EXTENSION_SETTINGS.md).

## Privacy highlights

- Your images and videos are processed **on your Mac**. Nothing is uploaded to a
  Smaller Please server.
- The original file is never modified; Smaller Please only writes optimized copies.
- No account, no telemetry, no tracking, and no reading of your chat or browsing activity.
- Read the full statement in [`docs/PRIVACY.md`](docs/PRIVACY.md).

## Download and install

1. Open the [Beta.3 releases page](https://github.com/wberry9813/Smaller-Please/releases) and
   download the installer for your Mac:
   `Smaller-Please-Installer-0.1.0-beta.3-macos-arm64.dmg`.
2. Open `Smaller Please Installer.app` and click **Install**, then follow the step-by-step guide
   in [`docs/DOWNLOAD.md`](docs/DOWNLOAD.md) to add the Chrome extension.

This is a Beta release. The install guide covers the one-time Chrome setup (Chrome requires
**Developer mode → Load unpacked**, which cannot be automated) and what to expect.

## Requirements

- macOS 12 or later on Apple Silicon (arm64).
- Google Chrome for the browser integration.

## Beta status and known limitations

- **Beta software** — features and behavior may change before a stable release.
- **Chrome Web Store listing is Planned.** Adding the extension requires the manual
  **Developer mode → Load unpacked** step.
- A **public Homebrew tap is Planned**; there is no public tap today.
- **Automatic updates are Planned.** Update by running the newer installer and reloading the
  extension; see [`docs/update.md`](docs/update.md).
- A **public Media Pack download service is Planned**; the Media Pack is a separate local
  artifact only.
- **Intel (`x86_64`) Macs and Windows are not supported yet.**
- Some website upload paths are not yet live-verified, and a few are verified `passthrough`
  (the original is uploaded unchanged). These are labeled honestly in the extension; see
  [`docs/EXTENSION_SETTINGS.md`](docs/EXTENSION_SETTINGS.md).

## Documentation

- [Download and install](docs/DOWNLOAD.md)
- [Install on macOS](docs/install/macos.md) — installer and [Homebrew (Planned)](docs/install/homebrew.md)
- [Extension settings and website rules](docs/EXTENSION_SETTINGS.md)
- [CLI reference](docs/CLI.md)
- [Updating](docs/update.md) · [Uninstalling](docs/uninstall.md)
- [Privacy](docs/PRIVACY.md) · [Privacy policy page](PRIVACY.md)
- [Troubleshooting](docs/troubleshooting/browser-extension.md)

## License

MIT. See [`LICENSE`](LICENSE). Third-party components and licenses are listed in
[`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md).
