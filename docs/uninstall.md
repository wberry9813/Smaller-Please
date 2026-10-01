# Uninstall Guide

Uninstalling Smaller, Please involves two distinct boundaries. Because the tool integrates closely with your system and Chrome profile, the CLI helps clean up its own integration files safely.

## 1. Remove integration and program files

First, run the `smaller` uninstaller to safely clean up the staged extension, the Native Host, and the optional Media Pack:

```bash
smaller uninstall
```

**What this does:**
- It removes the program and integration files (Native Host launcher, staged extension in `~/Applications`).
- **By default, it preserves:** your config file, your cached/optimized media (`store`/`cache`), browser-local data, Homebrew, and your original user media. 

If you want to completely wipe your settings and cache as well, use the explicit flags:
```bash
smaller uninstall --remove-settings --remove-cache
```

*(Other options include `--incomplete` to clean up an interrupted installation, and `--dry-run` to preview the deletion plan.)*

## 2. Remove the Homebrew package

After cleaning up the integrations, remove the CLI executable:

```bash
brew uninstall smaller-please
```
This removes only the Homebrew-managed payload (the Cellar and linked bin/share directories). It does not touch your user profile.

## 3. Remove the extension from Chrome (Manual Step)

Google Chrome requires you to manually remove the extension from your profile:
1. Open `chrome://extensions`.
2. Find the **Smaller, Please** card.
3. Click **Remove** and confirm.
*(This is the only way to clear the browser-local extension storage and history.)*
