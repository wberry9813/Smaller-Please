# Download and install Smaller, Please

This page is the plain, step-by-step install for the macOS app. It takes about two minutes.

Smaller, Please compresses images and videos **on your Mac** before they are uploaded to AI
apps such as ChatGPT and Claude. Nothing is uploaded to a Smaller, Please server.

> **System requirements:** macOS 12 or later. The bundled media engine requires Apple Silicon
> (arm64). Google Chrome is needed for the browser integration.

## 1. Download the installer

1. Open the [Smaller, Please releases page](https://github.com/wberry9813/Smaller-Please/releases)
   and pick the version you want — the current beta is
   [v0.1.0-beta.2](https://github.com/wberry9813/Smaller-Please/releases/tag/v0.1.0-beta.2).
2. Download the installer DMG for your release. A beta release carries its beta number in the
   filename, e.g. `Smaller-Please-Installer-0.1.0-beta.2-macos-arm64.dmg`; a stable release is
   `Smaller-Please-Installer-0.1.0-macos-arm64.dmg`.
3. (Recommended) Download `SHA256SUMS` from the same release and check the file before you
   open it. A matching checksum means the download completed correctly:

   ```sh
   shasum -a 256 Smaller-Please-Installer-0.1.0-beta.2-macos-arm64.dmg
   ```

The Beta.2 DMG is **signed with a Developer ID Application certificate and notarized by
Apple**, with the notarization ticket stapled to the DMG; Gatekeeper accepts it. Only open
downloads from a source you trust.

## 2. Open the installer

1. Double-click the downloaded `.dmg` file. A window opens. It also contains a plain-text
   install guide, `README.txt`.
2. Double-click **Smaller Please.app** inside that window.
3. If macOS says the app cannot be opened because it is from an unidentified developer, this
   means the build you downloaded was not notarized. For the signed and notarized Beta.2 DMG
   this warning should not appear; if you do see it on an older build, right-click (or
   Control-click) the app and choose **Open**, then confirm.

## 3. Install Smaller, Please

1. In the installer, click **Install**.
2. Wait for the four steps to show **Ready**:
   - Smaller, Please Core
   - Media Engine
   - Native Host
   - Browser Extension Files
3. You do **not** need to type a password. The installer writes only inside your own user
   account — it never changes your system or another user's files.
4. When it says **Installation complete**, leave the installer open for the next step.

## 4. Open Chrome's extensions page

In the installer, click **Open Chrome Extensions** (or open Google Chrome and go to
`chrome://extensions`). This is the page that manages Chrome add-ons.

## 5. Turn on Developer mode

On the `chrome://extensions` page, turn on the **Developer mode** switch in the top-right
corner. The page will show extra buttons, including **Load unpacked**.

## 6. Load the Smaller, Please extension

1. On the installer, click **Open Folder** (or **Copy Path**) next to the Browser Extension
   section. The folder is:

   ```
   ~/Applications/Smaller Please Extension
   ```

2. On the `chrome://extensions` page, click **Load unpacked**.
3. In the file picker, select the `Smaller Please Extension` folder and confirm.
4. Smaller, Please now appears in your Chrome extensions list.

That's it. Open ChatGPT or Claude and attach, drop, or paste an image or video — Smaller,
Please will optimize it locally before it is uploaded, and it will fall back to the original
file if anything goes wrong.

## Why is there a manual Chrome step?

Chrome only lets extensions from the **Chrome Web Store** install with one click. Everything
else — including Smaller, Please in this release — must be added through
**Developer mode → Load unpacked**, where you point Chrome at a folder you chose yourself.

This is Chrome's security model for extensions, not a limitation of Smaller, Please. It means
you can always see exactly which folder you loaded, and you stay in control of updates. A
one-click Chrome Web Store listing is planned for a future release; until then, this manual
step is expected and required.

## Updating

Re-run the installer for a newer version, then return to `chrome://extensions` and click the
**reload** icon on the Smaller, Please card so Chrome picks up the updated files. Reloading is
the only step needed; you do not need to remove and re-add the extension. See
[`update.md`](update.md).

## Uninstalling

Open the installer and use the **Uninstall…** option, or ask Chrome to **Remove** the extension
on `chrome://extensions`. By default your settings and cached data are kept so a reinstall keeps
your preferences; you can opt in to removing them in the installer. See
[`uninstall.md`](uninstall.md) for the details.

## If something looks wrong

- The extension card does not say **Ready** in the installer: click **Repair** in the installer,
  then reload the extension in Chrome.
- The extension is listed in Chrome but nothing happens: click **Open Chrome Extensions**, make
  sure the extension is enabled, and click the reload icon on its card.
- More help: [`troubleshooting/browser-extension.md`](troubleshooting/browser-extension.md) and
  [`troubleshooting/native-host.md`](troubleshooting/native-host.md).

Smaller, Please never modifies your images or videos — it only reads them and writes optimized
copies. See [`PRIVACY.md`](PRIVACY.md).
