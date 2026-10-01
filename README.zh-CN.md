# Smaller, Please

**在上传前于本地准备更小的 AI 适用文件，同时保留原始文件。**

[阅读英文版 (Read in English)](README.md)

## 安装

Homebrew 是唯一受支持的安装方法（仅限 Apple Silicon (arm64) 上的 macOS 12+）。

```bash
brew tap wberry9813/smaller-please
brew install smaller-please
smaller setup
smaller doctor
```

然后请手动加载浏览器扩展（见下文）。

## 它是什么

- **图片优化：** HEIC、JPEG 和 TIFF 将被优化为更小的 JPEG 或 PNG 副本。
- **视频优化：** MP4、MOV 和 M4V 使用 H.264 + AAC 在本地进行压缩。
- **浏览器上传优化：** 拦截并缩小在受支持 AI 网站上的上传文件。
- **AI 代理准备：** CLI 工具输出机器可读的 JSON，以便代理使用优化后的文件。
- **本地处理：** 所有操作均在你的 Mac 上运行。不会将任何媒体发送到我们的服务器。
- **保留源文件：** 默认情况下绝不修改或删除原始文件。
- **Smart 与 Maximum 压缩：** “Smart”模式提供安全默认值，“Maximum”模式提供更激进的压缩（需要 Pro）。
- **元数据控制：** 显式移除全部元数据，使用 `--metadata remove-all`（需要 Pro）。

## 浏览器扩展

通过 Homebrew 安装后，你必须手动加载扩展：

1. 运行 `smaller setup` 以准备扩展文件。
2. 运行 `smaller extension path` 以显示确切的文件夹路径（`~/Applications/Smaller Please Extension`）。
3. 在 Google Chrome 中打开 `chrome://extensions`。
4. 开启**开发者模式 (Developer mode)**（右上角）。
5. 点击**加载已解压的扩展程序 (Load unpacked)**，然后选择第 2 步中输出的文件夹。

**受支持的网站：**
默认情况下，扩展仅在以下三个内置的 AI 网站上运行：
- **ChatGPT** (`chatgpt.com`)：支持点击和拖放上传。
- **Gemini** (`gemini.google.com`)：支持点击上传。拖放上传为 **passthrough（原样上传）**（原始文件会原样附加，不会被优化）。
- **DeepSeek** (`chat.deepseek.com`)：支持点击和拖放上传。

## CLI

`smaller` 命令帮助你优化媒体文件并检查系统运行状况。

```bash
smaller optimize ./photo.jpg
smaller optimize ./clip.mov
smaller prepare ./photo.jpg --json
smaller prepare-batch ./input --json
smaller doctor
```

- `optimize`：用于优化图片、视频或目录的单一命令。
- `prepare` / `prepare-batch`：输出包含 `use_path` 的冻结 JSON 计划。`use_path` 是你应该上传或使用的确切文件。

## 与 AI 代理一起使用

Smaller, Please 内置了面向 AI 代理的公开 Agent Skill。安装后运行：

```bash
smaller get skills
```

将输出内容复制给你的 AI 代理。输出中包含安装指令和内置 Skill，代理会根据当前环境完成 Skill 配置。

代理在准备文件时必须：

- 优先使用 `smaller prepare` / `smaller prepare-batch --json`；
- 使用 JSON 计划中返回的确切 `use_path`；
- 绝不自行构造输出路径；
- 绝不删除或覆盖原始源文件；
- 绝不静默替换为其他压缩工具；
- 除非环境真正执行了上传，否则不要声称已完成上传。

## Free 与 Pro

- **Free（免费版）：** 包含 `smart` 模式优化、浏览器优化以及默认元数据行为。
- **Pro（专业版）：** 通过本地验证的许可证密钥，增加 `maximum` 压缩模式和元数据移除（`remove-all`）功能。购买通道目前尚未普遍开放。

## 隐私

你的媒体处理完全在本地进行。Smaller, Please 不会将你的图片或视频上传到任何媒体处理服务器。阅读[权威隐私政策](PRIVACY.md)。

## 更新

通过 Homebrew 更新，然后刷新本地配置：

```bash
brew update
brew upgrade smaller-please
smaller setup
smaller doctor
```
升级后，前往 `chrome://extensions` 并点击 Smaller, Please 扩展卡片上的刷新/重载图标。

## 卸载

```bash
brew uninstall smaller-please
smaller uninstall
```
有关默认保留哪些内容的详细信息，请参阅[卸载指南](docs/uninstall.md)。

## 故障排除

请参阅有关
[浏览器扩展](docs/troubleshooting/browser-extension.md)、
[原生主机](docs/troubleshooting/native-host.md)、
[核心组件](docs/troubleshooting/core.md)、
[存储](docs/troubleshooting/storage.md) 和
[媒体引擎](docs/troubleshooting/media-engine.md) 的故障排除指南。

## 许可证

MIT. 参见 [`LICENSE`](LICENSE) 和 [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md)。

## 网站 / 发布版本

- **网站:** [https://smaller-please.inchmirror.studio](https://smaller-please.inchmirror.studio)
- **发布版本:** [https://github.com/wberry9813/Smaller-Please/releases](https://github.com/wberry9813/Smaller-Please/releases)
