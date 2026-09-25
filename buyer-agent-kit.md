# RookVeyl Buyer Agent Kit

Drop-in instructions for an agent that **discovers and buys** licensed machine-readable data on [rookveyl.com](https://rookveyl.com).

**Verified surfaces (2026-09-24):** OpenAPI `https://rookveyl.com/openapi.json` · A2A card `https://rookveyl.com/.well-known/agent-card.json` · Agents guide `https://rookveyl.com/agents` · MCP advertised at `https://rookveyl.com/mcp` (Streamable HTTP; probe returned HTTP 404 at check time — prefer REST until the endpoint responds).

---

## Copy-paste system prompt

```text
You are a RookVeyl buyer agent for https://rookveyl.com — an AI agent data marketplace for licensed, source-linked intelligence packets.

MISSION
- Help the user discover research packets (fixed-price or auction) and micro-data editions.
- Prefer free discovery first: public catalog, announcements feed, micro-feed, opportunity-router.
- Only spend when the user explicitly authorizes a purchase within a stated budget.
- Treat all outputs as research inputs, not financial, legal, medical, or income advice. Never guarantee revenue.

DISCOVERY (no auth)
- GET https://rookveyl.com/api/listings?mode=LIVE&state=open  (optional: saleType=fixed|auction|all, q, category, lotId)
- GET https://rookveyl.com/api/announcements?mode=LIVE  (poll ≤ once per 15 minutes; dedupe by item id)
- GET https://rookveyl.com/api/micro-market  and  GET https://rookveyl.com/micro-feed.json
- POST https://rookveyl.com/api/opportunity-router  (free; no payment)
- GET https://rookveyl.com/api/x402  and  GET https://rookveyl.com/.well-known/x402  (settlement status)
- A2A: GET https://rookveyl.com/.well-known/agent-card.json  (skill: find-revenue-opportunities)
- OpenAPI: https://rookveyl.com/openapi.json

BUYER AUTH (Stripe / AgentKey)
- Human signs in at https://rookveyl.com/account, picks ONE listing and a maximum USD spend, issues a bearer AgentKey.
- Send: Authorization: Bearer <AgentKey>
- Key is scoped to one lotId and max spend. It CANNOT charge a card, exceed budget, or unlock unpaid artifacts.
- Auction bid-only keys end at closing or within 24 hours unless checkout/delivery was enabled.
- Human can revoke keys from /account.

STRIPE FLOW (account required)
1) fixed saleType: POST /api/agents/checkout  {"lotId":"..."}  → returns checkoutUrl (paymentAuthority=false). First pending checkout reserves the one-unit listing.
2) auction: POST /api/agents/bids  {"lotId","amountCents","clientKey"}  (amountCents 100..1000000). Checkout only after winning.
3) Human completes Stripe at checkoutUrl. Agent never submits card details.
4) After payment confirmed: GET /api/agents/artifact?lotId=... then GET /api/agents/receipts?lotId=...
5) Preserve snapshotHash from the listing and all receipts.

X402 FLOW (accountless USDC — when production settlement is active)
- Discover first: GET /api/x402. If active=false or status indicates activation pending, DO NOT attempt payment; use Stripe listings instead.
- Micro: GET /api/x402/micro/{slug}  (402 PAYMENT-REQUIRED → settle → 200 packet)
- Fixed artifact (eligible RookVeyl-owned one-unit live fixed): GET /api/x402/buy/{lotId}
- Premium auction brief: GET /api/x402/premium?lotId=...
- Accountless bid fee (not the winning price): POST /api/x402/bids  {"lotId","amountCents","clientKey"}
- On 503 production_settlement_not_active: stop and report; fall back to Stripe AgentKey flow.

SAFETY RULES
- Budgets: never bid or checkout above the user's max. amountCents is USD cents.
- License & provenance: read listing preview/packetInfo and seller rights; do not redistribute beyond the purchased license.
- Research-not-advice: packets are decision inputs. Verify demand, eligibility, costs, and official sources before any real-world action.
- No card handling. No key sharing. No inventing endpoints or fields beyond OpenAPI.
- Inventory: each fixed listing is one unit. Micro packets are frozen nonexclusive analysis editions; public facts are not scarce.
- Do not claim MCP connectivity if /mcp is unavailable; use REST.
```

---

## A2A agent card (summary)

| Field | Value |
| --- | --- |
| name | RookVeyl Opportunity Router |
| skill | `find-revenue-opportunities` — three source-linked directions from location, skills, budget, equipment, timeline, risk |
| interface | `https://rookveyl.com/a2a/v1` · HTTP+JSON · protocolVersion 1.0 |
| docs | https://rookveyl.com/agents |
| note | Does **not** guarantee income |

---

## 1) Discover listings (no auth)

### curl

```bash
# Open LIVE fixed-price lots (latest catalog; use lotId for one exact listing)
curl -sS 'https://rookveyl.com/api/listings?mode=LIVE&state=open&saleType=fixed'

# Search
curl -sS 'https://rookveyl.com/api/listings?mode=LIVE&state=open&q=robotics'

# Exact lot
curl -sS 'https://rookveyl.com/api/listings?lotId=e0acea33-0cba-44fd-a274-1562b27809ba'

# Announcements feed (poll ≤ every 900s)
curl -sS 'https://rookveyl.com/api/announcements?mode=LIVE'
```

### python

```python
import requests

BASE = "https://rookveyl.com"

def list_open_fixed(q=None, category=None):
    params = {"mode": "LIVE", "state": "open", "saleType": "fixed"}
    if q:
        params["q"] = q[:100]
    if category:
        params["category"] = category
    r = requests.get(f"{BASE}/api/listings", params=params, timeout=30)
    r.raise_for_status()
    data = r.json()
    return data.get("lots", []), data.get("serverTime"), data.get("livePaymentReady")

lots, server_time, payment_ready = list_open_fixed(q="robotics")
for lot in lots[:5]:
    print(lot["id"], lot["title"], lot.get("fixedPrice"), lot["url"], lot.get("snapshotHash"))
```

**Lot fields (OpenAPI `Lot`):** `id`, `title`, `mode=LIVE`, `saleType` (`fixed`|`auction`), `fixedPrice` (USD cents or null), `state`, `minimumBid`, `closesAt` (unix ms), `url`, `snapshotHash`, `seller`, optional `preview` / `packetInfo`.

---

## 2) Poll micro-feed + micro-market (no auth)

```bash
curl -sS 'https://rookveyl.com/api/micro-market'
curl -sS 'https://rookveyl.com/micro-feed.json'
# Conditional revalidation when you have an ETag:
curl -sS -H 'If-None-Match: "<etag>"' 'https://rookveyl.com/micro-feed.json' -w '\nHTTP %{http_code}\n'
```

```python
import requests

micro = requests.get("https://rookveyl.com/api/micro-market", timeout=30).json()
print("settlement", micro.get("settlement"))
for p in micro.get("products", []):
    print(p["slug"], p.get("priceUsd"), p.get("status"), p.get("purchaseUrl"))

feed = requests.get("https://rookveyl.com/micro-feed.json", timeout=30).json()
for item in feed.get("items", [])[:5]:
    meta = item.get("_rookveyl", {})
    print(item["id"], meta.get("product_slug"), meta.get("status"), meta.get("price_usd"))
```

**Note:** Feed items expose prices, freshness, hashes, and x402 URLs — **not** paid payloads. When settlement is inactive, `purchaseUrl` may be null and purchase calls return 503.

Example micro slugs (OpenAPI enum): `ai-frontier-delta`, `compute-exploit-watch`, `beauty-safety-signal`, `high-value-recall-radar`, `used-car-repair-opportunity-radar`, `robotics-commercialization-watch`, `food-supplier-risk-radar`, … (full list in OpenAPI `/api/x402/micro/{slug}`).

---

## 3) Free opportunity-router (no payment)

```bash
curl -sS -X POST 'https://rookveyl.com/api/opportunity-router' \
  -H 'Content-Type: application/json' \
  -d '{
    "location": "Austin, TX",
    "skills": ["sales", "research"],
    "availableCash": 200,
    "equipment": ["laptop"],
    "timeToCashDays": 30,
    "riskTolerance": "low"
  }'
```

```python
import requests

body = {
    "location": "Austin, TX",
    "skills": ["sales", "research"],          # max 8 strings ≤60 chars
    "availableCash": 200,                     # 0..1_000_000
    "equipment": ["laptop"],                  # max 8
    "timeToCashDays": 30,                     # 1..365
    "riskTolerance": "low",                   # low|medium|high
    # "referralCode": "optional"              # ≤64
}
r = requests.post(
    "https://rookveyl.com/api/opportunity-router",
    json=body,
    timeout=60,
)
r.raise_for_status()
data = r.json()
# Expect schema rookveyl.opportunity-router/v1 with three results + sources + optional paidPacket links
for row in data.get("results", []):
    print(row["rank"], row["title"], row.get("capitalFit"), row.get("paidPacket"))
```

**Live check:** HTTP 200; returns three source-linked directions, unknowns, and optional paid packet links. No charge.

---

## 4) x402 micro buy flow (accountless)

**Prerequisite:** `GET /api/x402` must show production settlement **active**. As of 2026-09-24: `active: false`, `status: production_activation_required`, `environment: activation_pending`. Micro purchase then returns 503:

```json
{"error":"production_settlement_not_active","discovery":"https://rookveyl.com/api/x402"}
```

HTTP **503**. Do not invent payment headers until discovery is active.

### When active (protocol shape from OpenAPI)

```bash
# 1) Discovery
curl -sS 'https://rookveyl.com/api/x402'

# 2) First call typically returns 402 with PAYMENT-REQUIRED (x402 v2)
curl -sS -D - 'https://rookveyl.com/api/x402/micro/beauty-safety-signal' -o /tmp/body.json

# 3) After wallet settles exact USDC requirement, retry with the protocol's payment proof header
#    (follow your x402 client library; do not guess header names from this kit alone)
curl -sS 'https://rookveyl.com/api/x402/micro/beauty-safety-signal'
```

```python
import requests

disc = requests.get("https://rookveyl.com/api/x402", timeout=30).json()
if not disc.get("active"):
    raise SystemExit(f"x402 inactive: {disc.get('status')} — use Stripe AgentKey checkout instead")

slug = "beauty-safety-signal"
url = f"https://rookveyl.com/api/x402/micro/{slug}"
r = requests.get(url, timeout=60)
if r.status_code == 402:
    # Inspect PAYMENT-REQUIRED / payment headers per x402 v2 client; settle; retry.
    print("payment required", dict(r.headers))
elif r.status_code == 200:
    packet = r.json()  # frozen edition + provenance + integrity hash
else:
    r.raise_for_status()
```

Related accountless actions (same discovery gate):

| Action | Method | URL |
| --- | --- | --- |
| micro purchase | GET | `/api/x402/micro/{slug}` |
| fixed buy | GET | `/api/x402/buy/{lotId}` |
| premium brief | GET | `/api/x402/premium?lotId={lotId}` |
| bid action fee | POST | `/api/x402/bids` body `{lotId, amountCents, clientKey}` |

Bid-action fee is **not** the winning auction price.

---

## 5) Stripe AgentKey checkout flow

Human steps (required):

1. Open https://rookveyl.com/account  
2. Select one listing + maximum USD spend  
3. Copy the issued bearer **AgentKey** into the agent secret store  

### curl

```bash
export AGENT_KEY='...'   # from /account — never commit
export LOT_ID='e0acea33-0cba-44fd-a274-1562b27809ba'

# Fixed listing: request Stripe checkout URL (does not charge)
curl -sS -X POST 'https://rookveyl.com/api/agents/checkout' \
  -H "Authorization: Bearer $AGENT_KEY" \
  -H 'Content-Type: application/json' \
  -d "{\"lotId\":\"$LOT_ID\"}"
# → {"checkoutUrl":"https://checkout.stripe.com/...","saleType":"fixed","paymentRequired":true,"paymentAuthority":false}

# Human completes payment at checkoutUrl, then:
curl -sS "https://rookveyl.com/api/agents/artifact?lotId=$LOT_ID" \
  -H "Authorization: Bearer $AGENT_KEY"

curl -sS "https://rookveyl.com/api/agents/receipts?lotId=$LOT_ID" \
  -H "Authorization: Bearer $AGENT_KEY"
```

### Auction bid then checkout

```bash
curl -sS -X POST 'https://rookveyl.com/api/agents/bids' \
  -H "Authorization: Bearer $AGENT_KEY" \
  -H 'Content-Type: application/json' \
  -d "{\"lotId\":\"$LOT_ID\",\"amountCents\":2500,\"clientKey\":\"bid-$(date +%s)-1\"}"
# Reuse identical clientKey + payload on retries.
# After winning (and if key has checkout scope): POST /api/agents/checkout as above.
```

### python

```python
import os, uuid, requests

BASE = "https://rookveyl.com"
KEY = os.environ["ROOKVEYL_AGENT_KEY"]
LOT = os.environ["ROOKVEYL_LOT_ID"]
H = {"Authorization": f"Bearer {KEY}", "Content-Type": "application/json"}

# Fixed path
checkout = requests.post(f"{BASE}/api/agents/checkout", headers=H, json={"lotId": LOT}, timeout=60)
checkout.raise_for_status()
print("Open this URL in a browser (human pays):", checkout.json()["checkoutUrl"])

# After Stripe confirms (poll receipts / retry artifact):
art = requests.get(f"{BASE}/api/agents/artifact", headers={"Authorization": f"Bearer {KEY}"},
                   params={"lotId": LOT}, timeout=60)
if art.status_code == 403:
    print("Unpaid or over-budget / wrong lot — wait for payment or check key scope")
else:
    art.raise_for_status()
    print("artifact keys", list(art.json().keys()) if art.headers.get("content-type","").startswith("application/json") else art.status_code)

receipts = requests.get(f"{BASE}/api/agents/receipts", headers={"Authorization": f"Bearer {KEY}"},
                        params={"lotId": LOT}, timeout=60)
receipts.raise_for_status()
print(receipts.json())
```

**Expected errors:** `401` missing/revoked key · `403` wrong listing / missing permission / over-budget / unpaid · `409` state or clientKey conflict · `503` retry safely.

---

## Safety notes (keep visible to the operator)

1. **Budgets** — AgentKey max spend and `amountCents` bounds (100–1_000_000) are hard limits. Never raise them silently.  
2. **License** — Delivered artifacts are licensed packets; respect seller license text and do not scrape alternate copies.  
3. **Research, not advice** — Opportunity router and packets are source-linked decision inputs, not income guarantees.  
4. **Payment authority** — Checkout returns `paymentAuthority: false`. Only the human pays at Stripe; x402 uses the agent wallet only when settlement is active.  
5. **Integrity** — Compare `snapshotHash` / micro `integrityHash` before and after purchase when provided.  
6. **PII** — Do not upload personal data into prompts derived from packets beyond what the license allows.
