## Description:

Use BridgeNode for paid AI inference — OpenAI-compatible LLM chat completions and MCP access — without API keys, registration, or subscriptions. Pay per request with Solana USDC via the x402 (HTTP 402) micropayment protocol; BridgeNode sponsors transaction fees so agents only need USDC. Use when an agent needs LLM inference but has no provider API key, has a Solana USDC wallet, or prefers deterministic pay-per-request pricing over subscriptions.

This skill is ready for commercial/non-commercial use.

## Publisher:

[BridgeNode](https://bridgenode.cc)

### License/Terms of Use:

MIT-0

## Use Case:

AI agents needing LLM inference without provider API keys: pay per request with Solana USDC via x402. OpenAI-compatible chat completions, smart routing (auto/eco/premium), MCP access.

### Deployment Geography for Use:

Global — BridgeNode is a hosted public service (bridgenode.cc); the skill itself is a markdown instruction set with no geographic restrictions.

## Known Risks and Mitigations:

Risk: The skill instructs agents to make real USDC payments on Solana mainnet; a misconfigured wallet or a large prompt could spend more than intended.

Mitigation: The skill documents exact-scheme pricing (pay for input + max_tokens before processing) and the minimum charge ($0.002); agents should check the 402 amount before signing and cap max_tokens.

## Reference(s):

- [BridgeNode](https://bridgenode.cc)
- [Agent map (llms.txt)](https://bridgenode.cc/llms.txt)
- [Models & pricing](https://bridgenode.cc/v1/models)

## Skill Output:

**Output Type(s):** [Markdown instructions]

**Output Format:** [Markdown]

**Output Parameters:** [None]

**Other Properties Related to Output:** [None]

## Skill Version(s):

1.0.0

## Ethical Considerations:

Users should evaluate whether this skill is appropriate for their environment, review any generated or modified files before relying on them, and apply their organization's safety, security, and compliance requirements before deployment.
