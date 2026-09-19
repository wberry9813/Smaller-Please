# Install Smaller Please via Homebrew

> **Status: Planned — not available today.**
>
> There is **no public Smaller Please Homebrew tap** yet, so there is no working
> `brew install` command for this product. Do not run a tap command expecting it to work, and
> do not treat any tap URL as real until this page says otherwise.

## What is planned

- A **public** tap (`homebrew-smaller-please`) that installs a prebuilt macOS arm64 release
  artifact — no compiler toolchain required.
- The formula would install the CLI binaries and the production extension payload. It would
  **not** run `smaller setup`, register the Native Host, mutate `$HOME`, touch Chrome, or
  install FFmpeg.
- After a Homebrew install, the one-time setup would still be:

  ```bash
  smaller setup              # config + managed store + Media Engine + Native Host + extension staging
  smaller doctor             # read-only health report
  smaller native status
  ```

  Followed by the manual Chrome step (open `chrome://extensions`, enable **Developer mode**,
  click **Load unpacked**, and select `~/Applications/Smaller Please Extension`).

## For now

Use the installer app instead: see [`../DOWNLOAD.md`](../DOWNLOAD.md) or
[`macos.md`](macos.md). Homebrew is never required for the CLI or the extension.

## Uninstall note

If a Homebrew installation exists in the future, `brew uninstall smaller-please` would remove
only Homebrew-managed files (the Cellar tree and linked `bin/`/`share/`); it would not remove
your config, store/cache, browser storage, staged extension, or Native Host user files. See
[`../uninstall.md`](../uninstall.md).
