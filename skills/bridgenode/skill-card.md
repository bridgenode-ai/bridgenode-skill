## Description:

BridgeNode — anonymous pay-per-request LLM inference for agents. OpenAI-compatible endpoint + MCP access. No API keys. No registration. No subscriptions. No data collection — every request anonymous, nothing stored. Pay per request in Solana USDC via x402 (HTTP 402); fees sponsored, agents need only USDC. Use when an agent has no provider API key, holds Solana USDC, or wants deterministic privacy-preserving pay-per-request pricing.

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
