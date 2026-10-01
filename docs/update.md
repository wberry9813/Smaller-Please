# Update Guide

Updates for Smaller, Please are distributed through Homebrew. There is no automatic background updater.

## How to Update

1. Upgrade the package via Homebrew:
```bash
brew update
brew upgrade smaller-please
```

2. Refresh your staged extension and Native Host files:
```bash
smaller setup
```
*(This ensures that `~/Applications/Smaller Please Extension` is updated with the new payload.)*

3. Verify the system health:
```bash
smaller doctor
```

4. Reload the extension in Chrome:
- Open `chrome://extensions`.
- Find the **Smaller, Please** card.
- Click the refresh/reload icon (Chrome is never automatically closed or refreshed for you).

## Media Pack
If you are using the optional LGPL Media Pack, it remains a local artifact only. There is no online downloader or `smaller update` command.
