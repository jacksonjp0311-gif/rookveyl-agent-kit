# RookVeyl Buyer Agent Kit

See the full kit in this repository after the next update, or use the live OpenAPI at https://rookveyl.com/openapi.json and agents guide at https://rookveyl.com/agents.

**Quick start (no auth):**

```bash
curl -sS 'https://rookveyl.com/api/listings?mode=LIVE&state=open'
curl -sS -X POST 'https://rookveyl.com/api/opportunity-router' \
  -H 'Content-Type: application/json' \
  -d '{"location":"Austin, TX","skills":["research"],"availableCash":200,"equipment":["laptop"],"timeToCashDays":30,"riskTolerance":"low"}'
```

Full copy-paste buyer prompt, x402 notes, and Stripe AgentKey flows land in the next commit once the growth kit pack is complete.
