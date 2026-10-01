# macOS Installation

Homebrew is the primary and supported path for installing Smaller, Please on macOS.

## Requirements
- macOS 12 or later
- Apple Silicon (arm64)
- Google Chrome

## Installation Steps
Please see the [Homebrew Installation Guide](homebrew.md) for the exact step-by-step commands (`brew install smaller-please`, followed by `smaller setup`).

## What an Install Produces

When you install via Homebrew and run `smaller setup`, the following components are produced:

| Component | Path / Details |
|---|---|
| **Core CLI** | Installed via Homebrew (`smaller`). |
| **Native Host** | Placed in your user profile to facilitate communication between the CLI and the Chrome extension. |
| **Media Engine** | The optional LGPL Media Pack (installed manually if needed). |
| **Extension** | Staged in `~/Applications/Smaller Please Extension` (you must manually load this in Chrome). |
| **User Data** | Config and cached data stored in `~/Library/Application Support/SmallerPlease/`. |

## Verify the Installation

After finishing the setup and loading the extension in Chrome, run:
```bash
smaller doctor
```
This prints a health report checking the Native Host, CLI, extension connection, and Media Engine.

## Uninstall

If you wish to remove the program, see the [Uninstall Guide](../uninstall.md).

---

> **Historical Note: macOS Installer App**
> Earlier betas (through `v0.1.0-beta.5`) shipped with a signed `.dmg` installer application. That packaging method is now dormant and historical. Future releases do not ship a DMG. Do not use the graphical installer for current or future versions.
