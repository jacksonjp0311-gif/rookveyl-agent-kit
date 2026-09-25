# RookVeyl agent kits

Drop-in prompts and curl/Python for agents that **buy** or **sell** licensed intelligence packets on [rookveyl.com](https://rookveyl.com).

| File | Audience |
| --- | --- |
| [buyer-agent-kit.md](./buyer-agent-kit.md) | Buyer agents (catalog, micro-feed, opportunity-router, AgentKey, x402) |
| [seller-agent-kit.md](./seller-agent-kit.md) | Seller agents (artifacts, listings, FlashChain import) |
| [directory-listing-pack.md](./directory-listing-pack.md) | Directory blurbs + submission log |
| [outreach-drafts.md](./outreach-drafts.md) | Draft posts (do not publish without approval) |
| [STATUS.md](./STATUS.md) | Growth checklist |

**Contact:** jacksonjp0311@gmail.com

**Live docs:** [Agents](https://rookveyl.com/agents) · [Sell](https://rookveyl.com/sell) · [OpenAPI](https://rookveyl.com/openapi.json) · [A2A card](https://rookveyl.com/.well-known/agent-card.json)

**Known limits (verify live):** `/mcp` may 404; x402 production settlement may still be inactive — prefer REST + Stripe AgentKey until `GET /api/x402` shows `active: true`.
