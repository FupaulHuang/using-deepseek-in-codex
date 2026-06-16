# Installing deepseek-codex-plugin for OpenCode

## Prerequisites

- [OpenCode.ai](https://opencode.ai) installed

## Installation

Add to the `plugin` array in your `opencode.json`:

```json
{
  "plugin": ["deepseek-codex@git+https://github.com/YOUR_USER/using-deepseek-in-codex.git"]
}
```

Restart OpenCode. The skill `using-deepseek-in-codex` will be auto-discovered.

## Verification

Ask: "How do I configure Codex CLI with DeepSeek?" — the skill should load and guide setup.

## Tool Mapping

When the skill references Claude Code tools:
- `Task` / `Agent` subagents → OpenCode's native task dispatch
- `Bash` → shell tool
- `Write` / `Edit` → file editing tools
- `Skill` → OpenCode's native `skill` tool

## Manual Install

If the plugin manager doesn't work:

```bash
git clone https://github.com/YOUR_USER/using-deepseek-in-codex.git ~/.config/opencode/deepseek-codex-plugin
```

Then add to `opencode.json`:
```json
{
  "skills": {
    "paths": ["~/.config/opencode/deepseek-codex-plugin/skills"]
  }
}
```
