---
name: using-deepseek-in-codex
description: Use when configuring Codex CLI to use DeepSeek models, or when Codex returns 404/401 errors against a DeepSeek endpoint. Covers the Moon Bridge proxy setup — the required translation layer between Codex's Responses API and DeepSeek's Anthropic-format API.
---

# Using DeepSeek in Codex CLI

## Overview

Codex CLI speaks **only the OpenAI Responses API** (`wire_api = "responses"`). DeepSeek's API speaks **Anthropic Messages** format. Direct connection is impossible — a protocol translation proxy is **required**.

```
Codex CLI  →  Moon Bridge (localhost:38440)  →  DeepSeek API
Responses        Transform mode                  Anthropic Messages
```

## Why Direct Connection Fails

| What | Wire Format | Endpoint |
|------|-------------|----------|
| Codex CLI v0.139+ | Responses API | `POST /v1/responses` |
| DeepSeek `/anthropic` | Anthropic Messages | `POST /v1/messages` |
| DeepSeek `/v1` | Chat Completions | `POST /v1/chat/completions` |

- `wire_api = "chat"` is **rejected** by Codex 0.139+ (config parse error)
- Pointing `base_url` directly at DeepSeek with `wire_api = "responses"` hits a non-existent `/v1/responses` endpoint
- There is no `wire_api = "anthropic"` or `wire_api = "messages"` option

A proxy is **not optional** — it's the only working architecture.

## Setup Steps

### 1. Install Moon Bridge

```bash
git clone https://github.com/ZhiYi-R/moon-bridge.git
cd moon-bridge
```

Requires Go 1.25+ (`conda install -c conda-forge go`).

### 2. Create `config.yml`

```yaml
mode: "Transform"
server:
  addr: "127.0.0.1:38440"

models:
  deepseek-v4-pro:
    context_window: 1000000
    max_output_tokens: 384000
    default_reasoning_level: "high"
    supported_reasoning_levels:
      - effort: "high"
      - effort: "xhigh"
    supports_reasoning_summaries: true
    default_reasoning_summary: "auto"
    extensions:
      deepseek_v4:
        enabled: true

providers:
  deepseek:
    base_url: "https://api.deepseek.com/anthropic"
    api_key: "sk-your-deepseek-key"
    version: "2023-06-01"
    offers:
      - model: deepseek-v4-pro
      - model: deepseek-v4-flash

routes:
  moonbridge:
    model: deepseek-v4-pro
    provider: deepseek

defaults:
  model: moonbridge
  max_tokens: 65536
```

### 3. Start Moon Bridge

```bash
go run ./cmd/moonbridge -config config.yml
# → Transform server listening on 127.0.0.1:38440
```

### 4. Generate Codex Config

Let Moon Bridge generate both `config.toml` and `models_catalog.json`:

```bash
CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
MODEL=$(go run ./cmd/moonbridge -config config.yml -print-codex-model 2>/dev/null)
go run ./cmd/moonbridge -config config.yml \
  -print-codex-config "$MODEL" \
  -codex-base-url "http://127.0.0.1:38440/v1" \
  -codex-home "$CODEX_HOME" \
  > "$CODEX_HOME/config.toml"
```

This writes:
- `~/.codex/config.toml` — provider pointing to `http://127.0.0.1:38440/v1`
- `~/.codex/models_catalog.json` — model capabilities (context window, reasoning levels, tool support)

### 5. Set Auth (Placeholder)

Moon Bridge holds the real API key. Codex just needs a non-empty value:

```json
{"OPENAI_API_KEY": "moonbridge-no-auth-needed"}
```

### 6. Verify

```bash
codex doctor
# → 17 ok · 0 fail
```

## Common Mistakes

| Mistake | Why Wrong |
|---------|-----------|
| **Pointing Codex directly at DeepSeek** | Responses API ≠ Chat Completions. 404 on `/v1/responses`. |
| **Setting `wire_api = "chat"`** | Rejected by Codex 0.139+ as invalid config. |
| **`requires_openai_auth = true`** | Moon Bridge handles auth. Set to false or omit. |
| **Skipping `models_catalog.json`** | Without it, Codex uses fallback model metadata — degraded tool-call reliability and wrong context window. |
| **Using `-print-codex-config` with `2>&1`** | Log line from stderr mixes into config.toml, breaking TOML parse. Redirect stderr separately. |
| **Restarting Codex without Moon Bridge running** | `connection refused` on `127.0.0.1:38440`. Start Moon Bridge first. |

## Quick Reference

```bash
# Start proxy
go run ./cmd/moonbridge -config config.yml &

# Test proxy
curl http://localhost:38440/v1/models

# Test full pipeline
codex exec --model moonbridge "Hello"

# Regenerate config after changing config.yml
MODEL=$(go run ./cmd/moonbridge -config config.yml -print-codex-model 2>/dev/null)
go run ./cmd/moonbridge -config config.yml \
  -print-codex-config "$MODEL" \
  -codex-base-url "http://127.0.0.1:38440/v1" \
  -codex-home "$HOME/.codex" \
  > "$HOME/.codex/config.toml"
```
