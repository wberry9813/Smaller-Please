# Privacy

Smaller, Please is a **local-first** tool. It reduces image and video size on your own Mac
before those files are uploaded to a site you chose. This page states plainly what happens to
your data.

## What Smaller, Please does with your media

- **Images and videos are processed locally, on your Mac.** The Core (the `smaller` command and
  the part the browser talks to) reads the file, creates an optimized copy, and hands that copy
  back to the browser. The **original file is never modified**.
- **Your media is not uploaded to any Smaller, Please server.** There is no cloud processing
  and no Smaller, Please account. The optimized copy is only uploaded to the site you are using
  (for example ChatGPT or Claude) by your own action, exactly as before.
- **Site conversations and page content are not stored or read.** The browser integration
  watches for images/videos you pick, drop, or paste; it does not read, store, or transmit your
  prompts, the page's chat text, or the page content.
- **Chrome history is not read.** Smaller, Please does not request or use the `history`
  permission and does not look at the pages you visited.
- **Your browser profile is not modified.** No command and no installer touches your Chrome
  profile, cookies, or saved passwords. Loading the extension is a manual Chrome step you
  perform yourself.
- **The Native Host only performs local communication.** The small helper process Chrome starts
  exists to pass a request from the browser extension to the local Core and the result back
  again, on your machine. It does not send anything to the network.

The browser extension requests only the `nativeMessaging` and `storage` permissions. To detect
an image or video before it uploads, it runs on normal websites (`http`/`https`); it does not
read page content, chat text, cookies, or your browsing history.

## What is stored locally

Everything below stays on your Mac and is only used to make the tool work and remember your
preferences:

- **Configuration** — your thresholds and settings, for example the image/video minimum size,
  the minimum-savings percentage, and the store location
  (`~/Library/Application Support/SmallerPlease/config.json` on macOS).
- **Managed store and cache** — optimized derivatives plus a cache of already-processed media
  and the temporary files used while optimizing, under the configured store directory (by
  default `~/.contextslim-bridge/store`). You can inspect and clean this with `smaller clean`.
- **Browser-local counters and history** — the extension stores byte counters and a short list
  of recent activity in Chrome's own local storage (`chrome.storage.local`), for example the
  number of bytes saved, a coarse site identifier, and the input method (picker, drop, or
  paste). The history is sanitized: it does not store filenames, paths, URLs, prompts, or file
  contents. You can clear it and reset the counters from the extension popup.
- **Installed program files** — the app, the Core, the Media Engine, and the extension files,
  under `~/Library/Application Support/SmallerPlease/` and
  `~/Applications/Smaller Please Extension`. These are program files, not your data.

`smaller uninstall` removes the program and integration files and, by default, **keeps** your
configuration and managed store/cache. Removing those is an explicit opt-in.

## What is not collected

- **No telemetry** and no analytics. Nothing about your usage is sent to us or to any third
  party.
- **No media content, no chat content, no browsing history, no page URLs** — not stored by
  Smaller, Please and not transmitted anywhere except to the site you are already using.
- **No account and no identifiers.** Smaller, Please does not create a user account or a
  tracking identifier.

## Network activity

Normal optimization makes no network request: the extension talks only to the local Native Host,
and the Core talks only to the local media engine. The only network activity that can happen
elsewhere is opt-in and unrelated to your media:

- `smaller setup` may offer to install FFmpeg with Homebrew **only** on an interactive run and
  **only** after you explicitly agree; that uses Homebrew's own services.

## Questions

This page describes the implemented behavior of Smaller, Please 0.1.0. If you find a statement
here that does not match what the software does, that is a bug — please report it.
