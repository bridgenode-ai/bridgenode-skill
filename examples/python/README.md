# BridgeNode Python example

Buyer-side example: AI agent pays for and calls BridgeNode with the official
x402 Python SDK (`x402[svm]`).

## Setup

```bash
pip install "x402[svm]" python-dotenv
cp .env.example .env
# fill SVM_PRIVATE_KEY (agent's Solana keypair, must have USDC ATA)
```

## Run

```bash
python main.py
```

## What it does

1. `GET /v1/models` — free, lists models and prices.
2. `POST /v1/chat/completions` — x402 payment handled automatically:
   request → 402 → sign partial TX (USDC TransferChecked + Memo) →
   retry with `PAYMENT-SIGNATURE` → 200 + `PAYMENT-RESPONSE`.

## Notes

- Solana mainnet, real USDC. Minimum charge: $0.002 per request.
- Gas is sponsored by BridgeNode — the agent only needs USDC, no SOL.
- The agent must have an existing USDC ATA (associated token account).
- Funding: send USDC on Solana mainnet to the agent's own wallet — BridgeNode never holds balances; every request is paid individually via x402 (exact amount in the 402 response). The USDC ATA is derived from the agent's wallet address; no manual token account setup needed.
