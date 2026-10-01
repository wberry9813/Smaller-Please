# Browser Extension Settings

The Smaller, Please browser extension provides a transparent interface to manage optimization settings directly within Google Chrome.

## Global Switch and Website Modes

The extension features a single global **Smaller** switch to easily toggle optimization on or off. When the switch is enabled, optimization behavior follows your selected website mode:

1. **Only selected websites (Allowlist)**: Optimization only occurs on websites that have an explicitly enabled rule.
2. **Optimize on all websites**: Optimization occurs on every website unless explicitly disabled by a rule.

The fresh-install default is **Only selected websites** with the built-in AI websites enabled.

## Unified Website Rules

All website rules (built-in and custom) are managed in a single, unified list. You can add any custom website to this list. 

### Built-in AI Websites
There are exactly three built-in AI websites. Their capabilities are rigorously verified:

- **ChatGPT (`chatgpt.com`)**
  - Click upload: Supported
  - Drag-and-drop: Supported
  - Image: Supported
  - Video: Supported
- **Gemini (`gemini.google.com`)**
  - Click upload: Supported
  - Drag-and-drop: **Passthrough** (The drag path is verified to upload the original file unchanged. This is an accepted limitation, not a failure. It is not optimized.)
  - Image: Supported
  - Video: Supported
- **DeepSeek (`chat.deepseek.com`)**
  - Click upload: Supported
  - Drag-and-drop: Supported
  - Image: Supported
  - Video: **Unverified** (Not tested; must not be assumed supported.)

## Supported File Types

The extension intercepts the following file types based on their extension:
- **Images**: `jpg`, `jpeg`, `png`, `webp`, `heic`, `heif`
- **Videos**: `mp4`, `mov`, `m4v`

Unsupported files are always left untouched and handled naturally by the website.

## Extension Lifecycle Distinction

Understanding the extension's status requires distinguishing between three states:
1. **Staged**: The extension files exist on your disk (`smaller setup` puts them there).
2. **Loaded**: Google Chrome has loaded the extension (you clicked **Load unpacked** in Developer mode).
3. **Connected**: The extension is actively communicating with the Native Host. 

Core commands cannot automate the "Loaded" step in your Chrome profile. You must manually perform the Developer mode **Load unpacked** step. 

## Export / Import Config

You can export your configuration as human-readable JSON. This is convenient for backup, sharing, or allowing an AI assistant to analyze it. You can import JSON back into the extension.

## Free / Pro Lock Hints

Under the **Compression** settings in the extension:
- **Maximum compression** and **Remove all metadata** are Pro features. 
- When running on a Free plan (without a valid licence), these controls remain visible but are disabled with a concise Pro lock hint. 
- The Core CLI remains the ultimate authority on whether Pro capabilities are granted.
