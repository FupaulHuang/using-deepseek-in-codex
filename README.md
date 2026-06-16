# DeepSeek + Codex CLI Plugin

Let Codex CLI talk to DeepSeek models via Moon Bridge.

Compatible with **Claude Code**, **Codex CLI**, **OpenCode**, and **Gemini CLI (Hermers)**.

## Why?

Codex CLI (v0.139+) speaks **only** the OpenAI Responses API. DeepSeek's API speaks **Anthropic Messages**. They're wire-incompatible — a translation proxy is **required**.

This plugin provides the setup skill and instructions to configure:

```
Codex CLI  →  Moon Bridge (127.0.0.1:38440)  →  DeepSeek API
Responses        Transform mode                  Anthropic Messages
```

## Prerequisites

- Codex CLI (`npm install -g @openai/codex`)
- Go 1.25+ (`conda install -c conda-forge go`)
- DeepSeek API key ([platform.deepseek.com](https://platform.deepseek.com))
- [Moon Bridge](https://github.com/ZhiYi-R/moon-bridge)

## Installation

### Claude Code

```bash
claude plugins install https://github.com/YOUR_USER/using-deepseek-in-codex
```

Or from within Claude Code:

```
/plugin install using-deepseek-in-codex@YOUR_USER
```

### Codex CLI

```bash
codex plugin install https://github.com/YOUR_USER/using-deepseek-in-codex
```

### OpenCode

Add to `opencode.json`:

```json
{
  "plugin": ["deepseek-codex@git+https://github.com/YOUR_USER/using-deepseek-in-codex.git"]
}
```

### Gemini CLI (Hermers)

Install from the Gemini extension registry, or manually:

```bash
git clone https://github.com/YOUR_USER/using-deepseek-in-codex.git
```

Then add the extension path to your Gemini CLI config.

## Quick Start

```bash
# 1. Clone Moon Bridge
git clone https://github.com/ZhiYi-R/moon-bridge.git
cd moon-bridge

# 2. Create config.yml with your DeepSeek key (see skill for template)

# 3. Start Moon Bridge
go run ./cmd/moonbridge -config config.yml &

# 4. Generate Codex config
CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
MODEL=$(go run ./cmd/moonbridge -config config.yml -print-codex-model 2>/dev/null)
go run ./cmd/moonbridge -config config.yml \
  -print-codex-config "$MODEL" \
  -codex-base-url "http://127.0.0.1:38440/v1" \
  -codex-home "$CODEX_HOME" \
  > "$CODEX_HOME/config.toml"

# 5. Verify
codex doctor
```

## Skill Reference

Ask any connected agent: *"configure Codex CLI with DeepSeek"* — the `using-deepseek-in-codex` skill loads automatically and guides the setup step by step.

## License

MIT
