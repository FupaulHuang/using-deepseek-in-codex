# DeepSeek + Codex CLI Plugin

Let Codex CLI talk to DeepSeek models via Moon Bridge.

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

## Installation

```bash
claude plugins install https://github.com/YOUR_USER/using-deepseek-in-codex
```

Or from within Claude Code:

```
/plugin install using-deepseek-in-codex@YOUR_USER
```

## Skill Reference

Ask Claude Code: "configure Codex CLI with DeepSeek" — the `using-deepseek-in-codex` skill will be auto-loaded and guide the setup step by step.

## License

MIT
