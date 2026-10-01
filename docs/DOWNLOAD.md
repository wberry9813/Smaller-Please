# Install Smaller, Please

Homebrew is the supported installation method. 

## Install with Homebrew

1. Install the CLI and extension payload via Homebrew:
```bash
brew tap wberry9813/smaller-please
brew install smaller-please
```

2. Run setup to configure the tool and stage the browser extension:
```bash
smaller setup
smaller extension path
smaller doctor
```

3. Load the extension in Chrome (Manual Step):
- Open `chrome://extensions` in Google Chrome.
- Enable **Developer mode** in the top right corner.
- Click **Load unpacked**.
- Select the folder printed by the `smaller extension path` command (typically `~/Applications/Smaller Please Extension`).

## Update

To update Smaller, Please via Homebrew:
```bash
brew update
brew upgrade smaller-please
smaller setup
smaller doctor
```
After upgrading, go to `chrome://extensions` and click the refresh/reload icon on the Smaller, Please extension card.

## Uninstall Overview

Uninstalling consists of two boundaries:
1. `brew uninstall smaller-please` removes only the Homebrew-managed payload (the CLI executable).
2. `smaller uninstall` removes program/integration files, Native Host, and the staged extension. It keeps config, store/cache, browser-local data, Homebrew, and user media by default.

For complete uninstall details, see [Uninstall Guide](uninstall.md). Note that removing the extension from Chrome's `chrome://extensions` page must be done manually.

## Why the manual Chrome step?

Chrome's security model dictates that extensions can only be installed with one click if they are hosted on the Chrome Web Store. For extensions hosted elsewhere, users must explicitly install them via **Developer mode -> Load unpacked**. This ensures you maintain control over updates and always know exactly what folder you are loading. 

## Troubleshooting

- **Extension not staged:** If `smaller extension path` returns an error, run `smaller setup` first.
- **Extension not working:** Click the reload icon in `chrome://extensions` and verify it is enabled.
- Read more: [Browser Extension Troubleshooting](troubleshooting/browser-extension.md) and [Native Host Troubleshooting](troubleshooting/native-host.md).

---

> **Historical Note:** Earlier betas (through `v0.1.0-beta.5`) shipped with a signed macOS installer DMG. This path is strictly historical and is no longer supported or produced. Homebrew is the only supported installation method.
