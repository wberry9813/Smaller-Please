# Agent Skill

The **Smaller, Please** Agent Skill teaches autonomous AI coding agents how to locally optimize images and videos before consuming them or uploading them to AI tools.

## Why Use the Agent Skill?
When you ask an agent like OpenCode or Claude Code to "read this screenshot" or "prepare this video for ChatGPT," the agent might natively try to upload massive source files, consuming unnecessary context window budget, network bandwidth, and time. 

The Agent Skill equips the AI with the exact contract and workflows to use the `smaller` CLI locally. The agent will prepare an optimized version, preserving your original source file, and only consume the optimized derivative. 

## Supported Environments
The Skill is built on the shared Agent Skills open standard.
- OpenCode: Verified (controlled discovery test)
- Codex: Verified (controlled discovery test)
- Claude Code: Compatible via the shared Agent Skills standard (not live-verified in the environment where this guide was written)

## Installation

**Important Distinction:** You must have the `smaller` CLI installed via Homebrew to use the tool. Installing the Skill merely teaches the agent *how* to use the CLI; it does not install the `smaller` binary itself.

The primary way to configure your AI agent is to use the built-in CLI command. After installing Smaller, Please via Homebrew, run:

```bash
smaller get skills
```

Give the output to your AI agent. It contains the bundled Skill, its stable path, and the install guidance. The bundled Skill that ships with the Homebrew install is the **source of truth**.

### Preferred installation: a stable symlink

Point your agent's skills directory at the **bundled stable path** (the exact path is printed by `smaller get skills`), not at a versioned Cellar path:

```bash
mkdir -p ~/.agents/skills
ln -sfn "$(brew --prefix)/share/smaller-please/skills/smaller-please" ~/.agents/skills/smaller-please
```

Because the target is the stable Homebrew share path, `brew upgrade smaller-please` updates the bundled Skill and the **same symlink serves the new version automatically** — no re-install needed; a newly started or reloaded agent reads the updated Skill. Never symlink to a versioned Cellar path (for example `/opt/homebrew/Cellar/smaller-please/<version>/...`); it breaks after an upgrade.

If a previously **copied** Smaller, Please Skill already exists at `~/.agents/skills/smaller-please`, replace only that Smaller, Please destination with the symlink. Do not remove unrelated skills, and do not delete the bundled source.

Other ecosystems may add a secondary location that points at the canonical `~/.agents` Skill, for example:

```bash
mkdir -p ~/.claude/skills
ln -sfn ~/.agents/skills/smaller-please ~/.claude/skills/smaller-please
```

### Copy fallback

If your agent ecosystem cannot use symlinked skill directories, copy the Skill files from the bundled path instead. A copied Skill is a **snapshot**: it will not track upgrades, so re-run the copy after a future `brew upgrade smaller-please`.

### Verification
You can verify the CLI is healthy and ready for the agent by running:
```bash
smaller doctor
```

## Example User Requests
Once installed, you can simply ask your agent:
- *"Prepare this architecture diagram for a ChatGPT upload."*
- *"Optimize all the videos in the `./recordings` folder so they are small enough for my context."*

## How the Agent Works Under the Hood

The agent relies on `smaller prepare` or `smaller prepare-batch --json` to generate a frozen JSON plan. 
The JSON document includes a `status` (such as `optimized`, `skipped`, or `fallback`) and a `use_path`. 

**Safety Rules for Agents:**
- Agents are instructed to strictly consume the exact `use_path` returned.
- Agents will never delete or overwrite the original source file.
- Agents will not silently substitute another compression tool when Smaller, Please is requested.

## Updating or Removing the Skill

- **Symlink install (preferred):** `brew upgrade smaller-please` updates the bundled Skill, and the symlink serves it automatically on the next agent start/reload — no re-install step.
- **Copy install (fallback):** re-run the copy after each `brew upgrade smaller-please`, because a copied Skill is a snapshot.

To remove the Skill, delete only the Smaller, Please destination (leave unrelated skills and the bundled source untouched):

```bash
rm -f ~/.agents/skills/smaller-please   # use `rm -rf` if you installed a copy instead of a symlink
rm -f ~/.claude/skills/smaller-please
```
