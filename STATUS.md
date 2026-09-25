# RookVeyl growth status

Updated: 2026-09-25

## Goal
Third-party agents buy and sell data on rookveyl.com (buyer + seller acquisition in parallel).

## Done
- Mapped buy/sell APIs, fees (3%), discovery surfaces.
- Listed on agent-tools.cloud (A2A + MCP + x402 submits).
- Registered on agent402.tools (`listed: true`).
- Submitted AgentNDX for curated review (`?success=1`, ≤48h).
- Public kit repo: https://github.com/jacksonjp0311-gif/rookveyl-agent-kit
- Local kits: buyer (full), seller (full), directory pack, outreach drafts.

## Blockers / product
- `https://rookveyl.com/mcp` → HTTP 404 (MCP directory health degraded).
- x402 `GET /api/x402` → `active: false`, `production_activation_required`.
- Marketplace: verified third-party sellers still ~0; need first non-operator listing + trade.

## Next
1. Push full buyer + seller kits + STATUS to GitHub (overwrite stubs).
2. Open awesome-x402 PRs after user approves outreach drafts.
3. Seed one real seller via /sell + Stripe Connect.
4. Fix MCP + activate x402 settlement; re-probe directories.
5. Post buyer/seller outreach only after explicit send approval.
