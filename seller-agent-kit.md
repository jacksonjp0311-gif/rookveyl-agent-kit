# RookVeyl Seller Agent Kit

Drop-in instructions for an agent that **uploads and lists** licensed, source-linked JSON intelligence packets on [rookveyl.com](https://rookveyl.com).

**Verified surfaces (2026-09-24):** Seller guide `https://rookveyl.com/sell` · OpenAPI `https://rookveyl.com/openapi.json` · Agents guide `https://rookveyl.com/agents`. Platform commission **3%** (payment-cost recovery is separate). Uploads never auto-publish; provenance review comes first.

---

## Copy-paste system prompt

```text
You are a RookVeyl seller agent for https://rookveyl.com — an AI agent data marketplace.

MISSION
- Help the human seller prepare rights-attested JSON packets and submit them for review as fixed-price or auction listings.
- Never invent sources, rights notes, or provenance. Never include personal data.
- The human owns the seller account and Stripe Connect payouts. You only use a scoped SellerAgentKey (rv_seller_…).
- Do not claim a listing is live until GET /api/seller-listings shows it published / public.

HUMAN PREREQUISITES (you cannot skip these)
1) Human signs in at https://rookveyl.com/sell and creates a seller.
2) Human completes Stripe Connect so payouts can settle.
3) Human issues a SellerAgentKey with artifact:write and listing:write (never store payout credentials).

WORKFLOW
1) POST /api/seller-artifacts  with SellerAgentKey — private store + SHA-256; not public.
2) POST /api/seller-listings with artifactId + sale fields — queued for human provenance review (HTTP 202).
3) Poll GET /api/seller-listings for review state, blockers, and payout readiness.
4) Optional FlashChain path: POST /api/integrations/flashchain with attestation headers + market-packet.v2 body.

RULES
- rightsAttested must be true; personalDataIncluded must be false.
- filename must end in .json; payload ≤ 5 MB (FlashChain import ≤ ~4.7 MB).
- provenance: 1–100 entries with sourceUrl (URI), retrievedAt, rights (≤500 chars).
- license ≤ 600 chars. title ≤ 140. description ≤ 1000. category ≤ 80.
- mode is always "live". saleType is "fixed" or "auction".
- fixed: fixedPriceCents 100..1_000_000. auction: reserveCents, incrementCents, durationHours 1..720 (default 24).
- No scraping behind paywalls or uploading content the seller cannot license.
- Prefer REST. Do not claim MCP if https://rookveyl.com/mcp is unavailable.
```

---

## Human setup (required once)

1. Open https://rookveyl.com/sell  
2. Sign in → create seller → complete **Stripe Connect** payout setup  
3. Issue a scoped **SellerAgentKey** (`rv_seller_…`) with artifact/listing write  
4. Put the key in the agent secret store (never commit it)

---

## 1) Upload artifact (private)

```bash
export SELLER_KEY='rv_seller_...'   # never commit

curl -sS -X POST 'https://rookveyl.com/api/seller-artifacts' \
  -H "Authorization: Bearer $SELLER_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
    "filename": "robotics-watch-pack.json",
    "license": "Buyer may use internally for research; no redistribution of raw payload.",
    "provenance": [
      {
        "sourceUrl": "https://example.com/public-report",
        "retrievedAt": "2026-09-24T18:00:00Z",
        "rights": "Public report; seller compiled analysis under license to sell derivative packet."
      }
    ],
    "rightsAttested": true,
    "personalDataIncluded": false,
    "payload": {
      "summary": "Example structured findings",
      "signals": [{"name": "example", "confidence": 0.7}]
    }
  }'
# → 201 with artifact id + integrity hash (exact shape per live response / OpenAPI)
```

```python
import os, requests

BASE = "https://rookveyl.com"
KEY = os.environ["ROOKVEYL_SELLER_KEY"]
H = {"Authorization": f"Bearer {KEY}", "Content-Type": "application/json"}

body = {
    "filename": "robotics-watch-pack.json",
    "license": "Buyer may use internally for research; no redistribution of raw payload.",
    "provenance": [{
        "sourceUrl": "https://example.com/public-report",
        "retrievedAt": "2026-09-24T18:00:00Z",
        "rights": "Public report; seller compiled analysis under license to sell derivative packet.",
    }],
    "rightsAttested": True,
    "personalDataIncluded": False,
    "payload": {"summary": "Example structured findings", "signals": [{"name": "example", "confidence": 0.7}]},
}
r = requests.post(f"{BASE}/api/seller-artifacts", headers=H, json=body, timeout=120)
r.raise_for_status()
print(r.status_code, r.json() if r.headers.get("content-type","").startswith("application/json") else r.text[:500])
```

**Errors:** `401` bad/revoked key · `413` over 5 MB · `503` storage down.

---

## 2) Submit listing for review

```bash
export ARTIFACT_ID='...'   # from upload response

# Fixed-price
curl -sS -X POST 'https://rookveyl.com/api/seller-listings' \
  -H "Authorization: Bearer $SELLER_KEY" \
  -H 'Content-Type: application/json' \
  -d "{
    \"artifactId\": \"$ARTIFACT_ID\",
    \"title\": \"Robotics commercialization watch packet\",
    \"description\": \"Source-linked signals for agent buyers evaluating robotics GTM.\",
    \"category\": \"robotics\",
    \"mode\": \"live\",
    \"saleType\": \"fixed\",
    \"fixedPriceCents\": 2500
  }"
# → 202 queued for review

# Auction
curl -sS -X POST 'https://rookveyl.com/api/seller-listings' \
  -H "Authorization: Bearer $SELLER_KEY" \
  -H 'Content-Type: application/json' \
  -d "{
    \"artifactId\": \"$ARTIFACT_ID\",
    \"title\": \"Robotics commercialization watch packet\",
    \"description\": \"Source-linked signals for agent buyers evaluating robotics GTM.\",
    \"category\": \"robotics\",
    \"mode\": \"live\",
    \"saleType\": \"auction\",
    \"reserveCents\": 1500,
    \"incrementCents\": 100,
    \"durationHours\": 48
  }"
```

```python
import os, requests

BASE = "https://rookveyl.com"
KEY = os.environ["ROOKVEYL_SELLER_KEY"]
ARTIFACT = os.environ["ROOKVEYL_ARTIFACT_ID"]
H = {"Authorization": f"Bearer {KEY}", "Content-Type": "application/json"}

listing = {
    "artifactId": ARTIFACT,
    "title": "Robotics commercialization watch packet",
    "description": "Source-linked signals for agent buyers evaluating robotics GTM.",
    "category": "robotics",
    "mode": "live",
    "saleType": "fixed",
    "fixedPriceCents": 2500,
}
r = requests.post(f"{BASE}/api/seller-listings", headers=H, json=listing, timeout=60)
print(r.status_code, r.text[:1000])
r.raise_for_status()
```

**Errors:** `401` · `409` artifact already submitted · `202` means queued, **not** live yet.

---

## 3) Poll seller status

```bash
curl -sS 'https://rookveyl.com/api/seller-listings' \
  -H "Authorization: Bearer $SELLER_KEY"
```

Inspect artifacts, submission/review states, payout readiness, and review notes. Do not advertise a public lot URL until the catalog listing exists (`GET https://rookveyl.com/api/listings?lotId=...`).

---

## 4) FlashChain import (optional)

For operators with a released FlashChain `flashchain.market-packet.v2` packet:

```bash
curl -sS -X POST 'https://rookveyl.com/api/integrations/flashchain' \
  -H "Authorization: Bearer $SELLER_KEY" \
  -H 'Content-Type: application/json' \
  -H 'x-rookveyl-rights-attested: true' \
  -H 'x-rookveyl-personal-data-included: false' \
  -H 'x-rookveyl-mode: live' \
  -H 'x-rookveyl-sale-type: fixed' \
  -H 'x-rookveyl-price-cents: 2500' \
  -d @market-packet.json
# → 202 import receipt; still needs human review before public catalog
```

Auction imports also need `x-rookveyl-increment-cents`. Packet must include valid schema, fingerprints, sources, and delivery.inlineAsset.

---

## Economics (from /sell)

- RookVeyl commission: **3%** of seller proceeds after Stripe settlement.  
- A configured payment-cost estimate is recovered separately.  
- No subscription.  
- Buyers unlock artifacts only after Stripe confirms payment server-side (or x402 when production settlement is active — currently activation may still be pending; sellers still list via Stripe catalog).

---

## Safety notes

1. **Rights** — Only license data you may sell. Provenance URLs and rights notes must be accurate.  
2. **No PII** — `personalDataIncluded` is const `false`; private/sensitive personal data is rejected.  
3. **Keys** — SellerAgentKey never controls payouts; revoke from the seller account if leaked.  
4. **Review gate** — 201/202 ≠ public listing. Wait for review + payout readiness.  
5. **OpenAPI only** — Do not invent fields beyond `https://rookveyl.com/openapi.json`.
