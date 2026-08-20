---
name: bridgenode
version: 1.0.13
description: BridgeNode — x402 pay-per-request AI inference. OpenAI-compatible API + MCP server, Solana USDC, gas-free micropayments. No API keys. Free models included. Live prices: bridgenode.cc/v1/models. Use when an agent lacks a provider API key or wants privacy-preserving per-request AI inference pricing.
metadata:
  author: BridgeNode
  version: "1.0.13"
  url: https://bridgenode.cc
  repository: https://github.com/applefanaimail-blip/bridgenode-skill
  network: solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp
  currency: USDC
  openclaw:
    emoji: "⚡"
    homepage: https://bridgenode.cc
compatibility: Any OpenAI-compatible agent with x402 payment support; MCP clients (streamable-http)
---

# BridgeNode

## Overview (natural language)

BridgeNode is an AI inference service for agents: anonymous LLM access without API keys, without registration, without subscriptions. Agents get an OpenAI-compatible chat completions endpoint and pay as they go with Solana USDC micropayments via the x402 payment protocol (HTTP 402). Model prices are published per token on the public models endpoint; transaction fees are sponsored, so an agent only needs USDC in its own wallet. Works with any OpenAI-compatible agent, MCP clients, and x402-capable SDKs.


## ⚠️ Cost Warning (read first)

This skill **spends real money (USDC)** on every request via on-chain Solana USDC micropayments (x402). There are no refunds once the provider has responded. Check live prices at `GET /v1/models` before use, set `max_tokens` to control cost, and use client-side spending limits (`BRIDGENODE_MAX_PER_CALL`, `BRIDGENODE_DAILY_CAP`) if available.


BridgeNode is an AI inference bridge. Agents get an OpenAI-compatible endpoint and pay per request with Solana USDC via the x402 protocol. No API keys, no registration, no subscriptions. Transaction fees (SOL) are sponsored by BridgeNode — the agent only needs USDC in its wallet.

## When to Use

- The agent needs LLM inference (chat completions) but has no provider API key.
- Pay-per-request is preferred over monthly subscriptions.
- The agent has a Solana wallet with USDC (or can receive it) and supports x402 payments.
- Deterministic, transparent per-request pricing is required.
- MCP-based agents that need a paid inference tool.

## Endpoints

| Endpoint | Purpose |
|---|---|
| `https://bridgenode.cc/v1` | OpenAI-compatible API base URL |
| `https://bridgenode.cc/v1/models` | Public model list + prices (no auth) |
| `https://bridgenode.cc/v1/chat/completions` | Chat completions (POST) |
| `https://bridgenode.cc/mcp` | MCP server (streamable-http) |
| `https://bridgenode.cc/llms.txt` | Full agent install map |

## Models & Pricing

Prices are in USDC per token (6 decimals). Always fetch live prices from `GET /v1/models` — they are the single source of truth and are generated from server config (never hardcoded here — stale prices cost money).

**🆓 Free models (no payment, no API key, no registration):** `gpt-oss-20b` · `gpt-oss-120b` · `glm-4.7-flash` · `glm-4.5-flash` · `glm-4.6v-flash` (vision).

**Paid models (pay-per-request):** 29 eco + 4 premium — DeepSeek, GLM (Z.AI), Kimi (Moonshot), MiniMax. Full list with live prices: `GET https://bridgenode.cc/v1/models`.

Pricing model: **exact scheme** — the agent pays for `input tokens + max_tokens` **before** processing. If the model generates fewer than `max_tokens`, the agent still pays for `max_tokens` (this is the business model, not a bug). Minimum charge per request: 2000 atomic units = $0.002 USDC.

## Reasoning Models — Important

- Thinking/reasoning models generate **reasoning tokens that SHARE the `max_tokens` budget** with the answer.
- Use `max_tokens >= 200` — a too-small limit can be fully consumed by reasoning, producing an **EMPTY answer** (the model returned 200 with no content).
- **An empty answer is NOT refunded** — the service was provided (the provider returned 200). Increase `max_tokens` and purchase again.
- Prefer `stream: true` for long generations (non-stream is capped at 4096).
- If you use tools with a thinking model: you MUST return `reasoning_content` in the next turn, otherwise the API returns 400.

## Payment Flow (x402 V2, exact scheme)

1. Send the request without payment headers.
2. Server responds `402 Payment Required` with a `PAYMENT-REQUIRED` header (base64 JSON): price, `payTo` address, USDC mint, memo, recent blockhash.
3. Agent constructs a **partial transaction**: USDC `TransferChecked` (amount = required) + Memo instruction, signs with its own wallet. Fee payer is NOT signed by the agent.
4. Agent retries the request with `PAYMENT-SIGNATURE` header (base64 JSON payload with the signed transaction).
5. Server verifies the payment and processes the request (fees sponsored — gasless for the agent).
6. Response is `200` with `PAYMENT-RESPONSE` header (settlement receipt).

Key details:

- Network: `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` (Solana mainnet)
- Asset: USDC `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`
- The agent must have an existing USDC ATA (associated token account) for the mint.
- The agent does **not** need SOL — BridgeNode sponsors transaction fees.
- Use the official x402 SDKs (`@x402/svm`, `x402[svm]`) or any x402-capable client — they handle the 402 → sign → retry flow automatically.

## Funding

- Requires USDC on Solana mainnet (no API keys, no registration)
- The agent keeps USDC in its own wallet — BridgeNode never holds balances; every request is paid individually via x402 (exact amount quoted in the 402 response)
- The agent must have an existing USDC ATA (associated token account) for the mint — it is derived from the agent's wallet address; no manual token account setup needed
- Gasless: BridgeNode sponsor covers Solana fees
- Optional client-side spending limits (SDK, enforced locally before signing — not server balances): `BRIDGENODE_MAX_PER_CALL`, `BRIDGENODE_DAILY_CAP`

## Quick Start (curl)

Step 1 — get payment requirements:

```bash
curl https://bridgenode.cc/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4-flash","messages":[{"role":"user","content":"hello"}],"max_tokens":100}'
```

Response: `402` with `PAYMENT-REQUIRED` header (amount, payTo, memo).

Step 2 — sign the partial transaction with an x402-capable client (e.g. `x402-proxy` (npx x402-proxy), official SDK, or `pay` CLI) and retry:

```bash
curl https://bridgenode.cc/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "PAYMENT-SIGNATURE: <base64 payload>" \
  -d '{"model":"deepseek-v4-flash","messages":[{"role":"user","content":"hello"}],"max_tokens":100}'
```

Response: `200` with the completion and `PAYMENT-RESPONSE` header.

## Quick Start (Python, x402 SDK)

```python
# pip install "x402[svm]"
# Official x402 client handles the full 402 -> sign -> retry loop.
```

Use the official `x402` Python client (`x402[svm]`) configured with the agent's Solana keypair; point it at `https://bridgenode.cc/v1/chat/completions`. Payment is automatic.

## SDKs

- **Python SDK:** `pip install bridgenode-llm` (https://pypi.org/project/bridgenode-llm) — full toolkit: `pip install bridgenode`
- **CLI:** `pip install bridgenode-cli` (https://pypi.org/project/bridgenode-cli) — `bridgenode chat "Hello!"`
- **TypeScript SDK:** `npm i @bridgenode/llm` (https://www.npmjs.com/package/@bridgenode/llm)
- **MCP wrapper:** `npm i @bridgenode/mcp` (https://www.npmjs.com/package/@bridgenode/mcp)

All SDKs handle the x402 payment handshake automatically (402 → sign → retry → 200), with fail-closed spending limits (`BRIDGENODE_MAX_PER_CALL`, `BRIDGENODE_DAILY_CAP`).

## MCP Usage

- One-line install: `claude mcp add bridgenode -s user -- npx -y @bridgenode/mcp@latest`
- Server URL: `https://bridgenode.cc/mcp` (streamable-http)
- Tool: `chat_completions` (model, mode, messages, max_tokens)
- Payment: x402 handshake per tool call; prices are annotated in `tools/list` (`x-x402`) as an indication — always check the actual amount in the 402 response before signing.

## Request Options

- `model`: explicit model ID from `/v1/models` (e.g. `deepseek-v4-flash`).
- `mode`: smart routing — `auto` (complexity-based tier), `eco` (cheapest), `premium` (best). If both `model` and `mode` are sent, `model` wins.
- `max_tokens`: request cap (default 4096, clamped to model max). Non-stream requests are capped at 4096 — use `stream: true` for longer generations.
- `stream`: SSE streaming supported (`stream: true`).

## Errors

| Status | Meaning |
|---|---|
| 400 | Bad request (unknown model, invalid body, oversized non-stream max_tokens) |
| 402 | Payment required — see `PAYMENT-REQUIRED` header |
| 413 | Request body too large (limit 2 MB) |
| 429 | Too many requests (queue limit) |
| 503 | Service busy — retry with backoff |

All errors use the OpenAI error format: `{"error": {"message": ..., "type": ..., "code": ...}}`.

## Notes
- Security: ClawHub security audit Pass; VirusTotal scan clean (no engine findings).

- Discovery: `https://bridgenode.cc/.well-known/agent-card.json`, `/.well-known/mcp.json`, `/.well-known/ai-manifest.json`
- Listed on x402-list: https://x402-list.com/services/bridgenode
- Listed on x402-dev: https://www.x402dev.com/awesome-projects/
- Listed on nohumans.directory: https://nohumans.directory/l/f1f74751-9d5
- Listed on gold-402: https://github.com/Haustorium12/gold-402/blob/main/directory/learning.md
- ClawHub skill: https://clawhub.ai/bridgenode/skills/bridgenode
- Transaction fees are sponsored (gasless) — the agent only needs USDC in its own wallet.
- Refunds: if the provider fails before any content is delivered, the payment is refunded automatically (reverse USDC transfer).
