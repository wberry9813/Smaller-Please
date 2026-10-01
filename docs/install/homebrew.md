# Homebrew Installation

Homebrew is the supported way to install Smaller, Please.

## Requirements
- **macOS 12+** on Apple Silicon (**arm64**)
- The formula explicitly refuses Intel architecture.

## Installation

Run the following commands:
```bash
brew tap wberry9813/smaller-please
brew install smaller-please
```

### What does the Homebrew formula install?
The formula installs:
1. The prebuilt CLI archive containing `bin/smaller` and the legacy `contextslim` alias.
2. The browser extension payload in the Homebrew share directory.

**What it does NOT do:**
- It never builds from source.
- It never automatically runs `smaller setup`.
- It never installs FFmpeg.
- It never touches your `$HOME` directory or Google Chrome profile.
- You do **not** need to run `brew trust`.

## Post-Install Setup

Because the formula only drops the payload in the Homebrew prefix, you must manually run setup to configure your user environment:

1. Stage the extension and Native Host:
   ```bash
   smaller setup
   ```
2. Locate the extension directory:
   ```bash
   smaller extension path
   ```
   *(This prints the `~/Applications/Smaller Please Extension` path.)*
3. Verify your installation:
   ```bash
   smaller doctor
   ```
4. Load the extension into Chrome:
   - Go to `chrome://extensions` in Chrome.
   - Enable **Developer mode**.
   - Click **Load unpacked** and select the folder printed in step 2.

## Update

To update:
```bash
brew update
brew upgrade smaller-please
smaller setup
smaller doctor
```
*(After upgrading, always reload the extension in Chrome.)*

## Uninstall Boundary

`brew uninstall smaller-please` only removes the files managed by Homebrew (the CLI and shared payload). It does **not** remove the staged extension in `~/Applications`, your settings, or the Native Host. See [Uninstall Guide](../uninstall.md) for complete cleanup instructions.
