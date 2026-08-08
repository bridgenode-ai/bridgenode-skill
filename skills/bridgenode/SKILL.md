---
name: bridgenode
description: BridgeNode — anonymous pay-per-request LLM inference for AI agents. OpenAI-compatible endpoint + MCP access. No API keys, registration, or subscriptions; no data collection — nothing stored. Accepts Solana USDC micropayments via x402 (HTTP 402); fees sponsored, agents need only USDC. Use when an agent lacks a provider API key or wants privacy-preserving pay-per-request pricing.
metadata:
  author: BridgeNode
  version: "1.0.0"
  url: https://bridgenode.cc
  repository: https://github.com/applefanaimail-blip/bridgenode-skill
  network: solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp
  currency: USDC
compatibility: Any OpenAI-compatible agent with x402 payment support; MCP clients (streamable-http)
---

# BridgeNode

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

Prices are in USDC per token (6 decimals). Always fetch live prices from `GET /v1/models` — they are the single source of truth and are generated from server config.

| Model | Input / token | Output / token | Context window | Max output |
|---|---|---|---|---|
| `deepseek-v4-flash` | $0.00000020 | $0.00000040 | 1,048,576 | 8,192 |
| `groq-llama-3.3-70b` | $0.00000079 | $0.00000099 | 131,072 | 32,768 |
| `deepseek-v4-pro` | $0.00000085 | $0.00000170 | 1,048,576 | 8,192 |

Pricing model: **exact scheme** — the agent pays for `input tokens + max_tokens` **before** processing. If the model generates fewer than `max_tokens`, the agent still pays for `max_tokens` (this is the business model, not a bug). Minimum charge per request: 2000 atomic units = $0.002 USDC.

## Payment Flow (x402 V2, exact scheme)

1. Send the request without payment headers.
2. Server responds `402 Payment Required` with a `PAYMENT-REQUIRED` header (base64 JSON): price, `payTo` address, USDC mint, memo, recent blockhash.
3. Agent constructs a **partial transaction**: USDC `TransferChecked` (amount = required) + Memo instruction, signs with its own wallet. Fee payer is NOT signed by the agent.
4. Agent retries the request with `PAYMENT-SIGNATURE` header (base64 JSON payload with the signed transaction).
5. Server verifies, settles on-chain (fee payer = BridgeNode, gasless for the agent), then processes the request.
6. Response is `200` with `PAYMENT-RESPONSE` header (settlement receipt).

Key details:

- Network: `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` (Solana mainnet)
- Asset: USDC `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`
- The agent must have an existing USDC ATA (associated token account) for the mint.
- The agent does **not** need SOL — BridgeNode sponsors transaction fees.
- Use the official x402 SDKs (`@x402/svm`, `x402[svm]`) or any x402-capable client — they handle the 402 → sign → retry flow automatically.

## Funding

- Requires USDC on Solana mainnet (no API keys, no registration)
- USDC ATA is auto-created on first USDC deposit — no manual token account setup
- Gasless: BridgeNode sponsor covers Solana fees
- Optional spending caps: `BRIDGENODE_MAX_PER_CALL`, `BRIDGENODE_DAILY_CAP`

## Quick Start (curl)

Step 1 — get payment requirements:

```bash
curl https://bridgenode.cc/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4-flash","messages":[{"role":"user","content":"hello"}],"max_tokens":100}'
```

Response: `402` with `PAYMENT-REQUIRED` header (amount, payTo, memo).

Step 2 — sign the partial transaction with an x402-capable client (e.g. `x402curl`, official SDK, or `pay` CLI) and retry:

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

## MCP Usage

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

- Discovery: `https://bridgenode.cc/.well-known/agent-card.json`, `/.well-known/mcp.json`, `/.well-known/ai-manifest.json`
- Transaction fees are sponsored (gasless) — only USDC balance matters.
- Refunds: if the provider fails before any content is delivered, the payment is refunded automatically (reverse USDC transfer).
