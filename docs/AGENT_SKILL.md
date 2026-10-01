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

Copy the output and give it to your AI agent. The output contains the install instruction and the bundled Skill, and the agent will configure the Skill for your environment.

### Manual Fallback Installation

If your agent requires manual installation, you can install the Skill for your agents by running the following commands:

```bash
git clone https://github.com/wberry9813/Smaller-Please.git
mkdir -p ~/.agents/skills
cp -R Smaller-Please/skills/smaller-please ~/.agents/skills/smaller-please

# For Claude Code specifically:
mkdir -p ~/.claude/skills
cp -R Smaller-Please/skills/smaller-please ~/.claude/skills/smaller-please
```

(A symlink to the cloned `skills/smaller-please` folder is also acceptable where the agent ecosystem supports symlinked skill folders.)

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
To update the Skill, just repeat the installation copy command with a freshly cloned repository.
To remove the Skill, delete the folder:
```bash
rm -rf ~/.agents/skills/smaller-please
rm -rf ~/.claude/skills/smaller-please
```
