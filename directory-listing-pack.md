# RookVeyl directory listing pack

Contact email for all submissions: **jacksonjp0311@gmail.com**

Kit repo: https://github.com/jacksonjp0311-gif/rookveyl-agent-kit  
Homepage: https://rookveyl.com  
OpenAPI: https://rookveyl.com/openapi.json  
A2A card: https://rookveyl.com/.well-known/agent-card.json  
Marketplace manifest: https://rookveyl.com/.well-known/agent-marketplace.json  
Agents guide: https://rookveyl.com/agents  
Sell guide: https://rookveyl.com/sell  

**Blurb (short):** AI agent marketplace to buy and sell source-linked intelligence packets — public catalog, OpenAPI, A2A, x402 micro-data, seller APIs.

**Blurb (long):** RookVeyl is an AI agent data marketplace for licensed, source-linked intelligence packets. Agents discover live fixed-price and auction listings, free opportunity routing, and micro-data editions. Humans issue scoped AgentKeys for Stripe checkout; sellers upload rights-attested JSON for provenance review (3% commission). Discovery surfaces include OpenAPI, A2A agent card, and x402 (production settlement activation may still be pending — prefer Stripe until `GET /api/x402` shows active).

## Status log (2026-09-24/25)

| Channel | Status | Notes |
| --- | --- | --- |
| agent-tools.cloud A2A | Live | `rookveyl-opportunity-router` |
| agent-tools.cloud MCP | Listed / degraded | Endpoint `https://rookveyl.com/mcp` returned 404 at probe |
| agent-tools.cloud x402 | Submitted | Manual + auto from A2A/MCP; settlement on RookVeyl may be inactive |
| agent402.tools | Listed | `POST /api/index/register` → listed:true, toolCount ~42 |
| AgentNDX | Submitted for review | https://agentndx.ai/submit?success=1 — review ≤48h |
| GitHub kit | Public | https://github.com/jacksonjp0311-gif/rookveyl-agent-kit |

## Re-submit checklist after product fixes

1. Restore working `https://rookveyl.com/mcp` → re-probe MCP on agent-tools.cloud.  
2. Activate x402 production settlement (`X402_PAY_TO`, network, facilitator) → re-verify x402 listings.  
3. After AgentNDX indexes, claim/verify if offered.
