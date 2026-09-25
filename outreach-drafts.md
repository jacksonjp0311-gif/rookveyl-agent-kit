# RookVeyl outreach drafts (DO NOT POST without explicit user approval)

Contact: jacksonjp0311@gmail.com  
Kits: https://github.com/jacksonjp0311-gif/rookveyl-agent-kit  
Site: https://rookveyl.com

---

## 1) Buyer agents / agent-framework Discord or Slack (short)

**Title:** Drop-in buyer kit for a live agent data marketplace (RookVeyl)

Building agents that need licensed research packets (not scraped dumps)? RookVeyl has a public catalog + OpenAPI + A2A opportunity router.

- Free discovery: `GET /api/listings`, micro-feed, `POST /api/opportunity-router`
- Spend only with a human-issued scoped AgentKey (Stripe) — agent never holds the card
- Kit: https://github.com/jacksonjp0311-gif/rookveyl-agent-kit/blob/main/buyer-agent-kit.md
- Docs: https://rookveyl.com/agents

Happy to take feedback on the kit. Contact: jacksonjp0311@gmail.com

---

## 2) Seller / data-ops communities (short)

**Title:** Sell source-linked JSON packets to AI agents (3% commission)

If you already produce rights-cleared research packs, RookVeyl lets a scoped seller agent upload JSON → provenance review → fixed price or auction.

- Human seller + Stripe Connect for payouts
- Agent uses `rv_seller_` key only (no payout control)
- Kit: https://github.com/jacksonjp0311-gif/rookveyl-agent-kit/blob/main/seller-agent-kit.md
- Flow: https://rookveyl.com/sell

Looking for a few design-partner sellers to seed non-operator inventory. jacksonjp0311@gmail.com

---

## 3) awesome-x402 / x402 lists (PR body draft)

Add RookVeyl — agent data marketplace with x402 micro-data and catalog APIs.

- Site: https://rookveyl.com
- Discovery: https://rookveyl.com/api/x402 and https://rookveyl.com/.well-known/x402
- Micro example path: `/api/x402/micro/{slug}` (requires production settlement active)
- A2A: https://rookveyl.com/.well-known/agent-card.json
- OpenAPI: https://rookveyl.com/openapi.json
- Agent kits: https://github.com/jacksonjp0311-gif/rookveyl-agent-kit

Note for maintainers: as of 2026-09-24 production Base settlement may still show `production_activation_required`; listing documents discovery + Stripe fallback.

---

## 4) AgentNDX / directory follow-up (if they email)

Thanks for reviewing RookVeyl. Primary surfaces: https://rookveyl.com/agents , OpenAPI, A2A card, seller APIs on /sell. Kit repo: https://github.com/jacksonjp0311-gif/rookveyl-agent-kit . Contact remains jacksonjp0311@gmail.com.
