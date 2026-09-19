# Smaller Please

**让上传更小。**

Smaller Please 会在图片和视频上传到 ChatGPT、Claude 等 AI 应用之前，在你的 Mac 上完成压缩。
所有处理都在本地进行，没有云端处理、没有账户、也没有遥测。

[下载 Beta 版](https://github.com/wberry9813/Smaller-Please/releases/tag/v0.1.0-beta.3) ·
[安装指南](docs/DOWNLOAD.md) · [隐私](docs/PRIVACY.md) · [English](README.md)

## 它是什么

当 AI 工具只需要一份视觉上准确的副本时，大尺寸截图、照片和导出的界面图片会浪费大量上传流量。
Smaller Please 会为每张图片或每个视频生成一份更小、便于 AI 使用的副本，并报告节省的字节数，
同时不会改动你的原始文件。

它既可以在命令行中使用，也集成了 Chrome：你在任何网站（例如 ChatGPT 或 Claude）中选择、拖入或
粘贴的图片和视频，会在上传前完成优化。

## 功能

- **本地图片压缩** —— HEIC/JPEG/TIFF → JPEG，并支持 PNG；带有“绝不增大”保护与最小节省比例
  阈值。
- **本地视频压缩** —— MP4/MOV/M4V → MP4（H.264 + AAC），同样带有“绝不增大”保护。
- **Chrome 集成** —— 优化你选择、拖入或粘贴到网站（例如 ChatGPT 或 Claude）中的图片与视频。
- **一个开关、两种网站模式与按网站规则** —— 默认仅在内置的 AI 网站白名单（ChatGPT、Claude、
  Gemini、DeepSeek、Perplexity、Grok、Microsoft Copilot、Poe、Mistral Le Chat）上运行；你随时
  可以切换到“在所有网站”模式，并通过一份统一的规则列表管理内置网站和你添加的任何网站。
- **可导出、便于阅读的配置** —— 把你的规则导出为纯 JSON，方便你（或 AI 助手）阅读与保存。
- **绝不增大你的文件** —— 如果优化后的副本没有更小，它会被丢弃，并使用原始文件。
- **没有云端处理** —— 文件始终留在你的 Mac 上。
- **macOS 安装程序** —— 用户级安装，无需输入密码。Beta.3 的 DMG 已使用 Developer ID 签名，
  并通过 Apple 公证/装订（stapled），因此 Gatekeeper 会接受它。

原始文件永远不会被修改或覆盖。

## 控制优化范围

打开扩展弹窗即可切换唯一的 **Smaller** 开关，并查看当前网站的实际状态。打开**设置**页可使用完整
控制：

- 用 **Smaller** 开关在所有网站开启或关闭优化——你的模式和网站规则都会保留。
- 选择**网站模式**：在所有网站优化，或仅优化选定的网站。
- 在**网站规则**列表中管理**内置**的 AI 网站（ChatGPT、Claude、Gemini、DeepSeek、Perplexity、
  Grok、Microsoft Copilot、Poe、Mistral Le Chat）和**自定义**网站，或添加任何网站。
- 将配置**导出**为可读的 JSON，并**导入**回来（或粘贴一份）。

Smaller 拥有广泛的网站访问权限，这样你添加自定义网站时无需每次都弹出新的权限提示。**默认情况下它
只在“仅选定的网站”（内置 AI 白名单）上运行**；你可以随时切换到“在所有网站优化”。优先级为：
开关 > 网站规则 > 模式，因此在白名单模式下未列出的网站不会被优化。它只会查看你选择、拖入或粘贴的
文件。

尚未通过实站验证的上传路径，其网站行会标记为**“未验证”**——这表示 Smaller 尚未在该网站上验证支持，
**绝不是**权限限制。网站行也可能带有 **`passthrough`（原样上传）** 能力：该上传方式经验证会原样上传
原始文件，这是可接受的限制，而不是失败。**在 Gemini 上，点击上传会被优化；拖放则会原样上传原始文件，
不进行优化。**全新安装的默认值是内置 AI 白名单，但已有配置绝不会被悄悄修改。只有确认已将文件交给
网站自身的上传路径后，才会报告优化成功；若无法确认，则改为上传原文件。详见
[`docs/EXTENSION_SETTINGS.md`](docs/EXTENSION_SETTINGS.md)。

## 隐私亮点

- 你的图片和视频**在你的 Mac 上**处理，不会上传到 Smaller Please 服务器。
- 原始文件永远不被修改；Smaller Please 只写出优化后的副本。
- 没有账户、没有遥测、没有跟踪，也不会读取你的聊天或浏览活动。
- 完整说明见 [`docs/PRIVACY.md`](docs/PRIVACY.md)。

## 下载与安装

1. 打开 [Beta.3 版本发布页面](https://github.com/wberry9813/Smaller-Please/releases)，下载适用于
   你的 Mac 的安装程序：
   `Smaller-Please-Installer-0.1.0-beta.3-macos-arm64.dmg`。
2. 打开 `Smaller Please Installer.app` 并点击 **Install**，然后按照
   [`docs/DOWNLOAD.md`](docs/DOWNLOAD.md) 中的分步指南添加 Chrome 扩展。

这是一个 Beta 版本。安装指南涵盖了只需一次的 Chrome 设置步骤（Chrome 要求使用
**开发者模式 → 加载已解压的扩展程序**，此步骤无法自动完成），以及你可能会看到的情况。

## 系统要求

- macOS 12 或更高版本，Apple Silicon（arm64）。
- 浏览器集成需要 Google Chrome。

## Beta 状态与已知限制

- **Beta 软件** —— 功能与行为在正式版之前可能发生变化。
- **Chrome 应用商店上架为计划中。** 添加扩展需要手动执行**开发者模式 → 加载已解压的扩展程序**。
- **公开的 Homebrew tap 为计划中**；目前没有公开 tap。
- **自动更新为计划中。** 更新方式是运行较新的安装程序并重新加载扩展；见
  [`docs/update.md`](docs/update.md)。
- **公开的 Media Pack 下载服务为计划中**；Media Pack 目前只是独立的本地制品。
- **暂不支持 Intel（`x86_64`）Mac 与 Windows。**
- 部分网站的上传路径尚未通过实站验证，少数为经验证的 `passthrough`（原样上传）。扩展会如实标注，
  见 [`docs/EXTENSION_SETTINGS.md`](docs/EXTENSION_SETTINGS.md)。

## 文档

- [下载与安装](docs/DOWNLOAD.md)
- [在 macOS 上安装](docs/install/macos.md) —— 安装程序与 [Homebrew（计划中）](docs/install/homebrew.md)
- [扩展设置与网站规则](docs/EXTENSION_SETTINGS.md)
- [CLI 参考](docs/CLI.md)
- [更新](docs/update.md) · [卸载](docs/uninstall.md)
- [隐私](docs/PRIVACY.md) · [隐私政策页](PRIVACY.md)
- [疑难解答](docs/troubleshooting/browser-extension.md)

## 许可证

MIT。见 [`LICENSE`](LICENSE)。第三方组件与许可证见
[`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md)。
