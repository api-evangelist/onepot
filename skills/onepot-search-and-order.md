---
name: Search and order make-on-demand molecules with onepot
description: >-
  Use the onepot API (via the first-party Python client) to search the onepot
  CORE make-on-demand compound space by structure, optionally decompose a target
  retrosynthetically with price/risk filters, and place a synthesis order.
api: https://www.onepot.ai/api
package: onepot (PyPI, pip install onepot, Python 3.11+)
source: https://www.onepot.ai/api
method: generated
generated: '2026-07-20'
operations:
- search
- search_stream
- order
---

# Search and order make-on-demand molecules with onepot

Operating instructions grounded in the documented onepot Python client methods
(https://www.onepot.ai/api). Every operation below is a real, published client
method; do not invent parameters beyond those documented.

## Prerequisites

- Install the client: `pip install onepot` (or `uv add onepot`). Requires Python 3.11+.
- Obtain an API key via "Request API access" on https://www.onepot.ai/api.
- Construct the client: `client = Client(api_key="your-api-key")`.
- Inputs are SMILES strings.

## 1. Similarity / substructure search (`search`)

Return ranked make-on-demand analogs with similarity scores and instant prices.

```python
from onepot import Client
client = Client(api_key="your-api-key")
resp = client.search(smiles_list=["c1ccc(NC(=O)c2ccccc2)cc1"], max_results=10)
for r in resp["queries"][0]["results"]:
    print(r["smiles"], r["similarity"], r["price_usd"])
```

## 2. Retrosynthesis + filters (`search`)

Decompose a target and filter hits by price, building-block availability, and
route risk.

```python
resp = client.search(
    smiles_list=["O=C(Nc1ccc(F)cc1)c1cccnc1"],
    decompose=True,
    max_results=25,
    max_price_usd=200,
    max_risk=0.3,
)
for r in resp["queries"][0]["results"]:
    print(r["smiles"], "from", len(r["building_blocks"]), "BBs")
```

## 3. Stream long-running searches (`search_stream`)

For long queries, consume server-sent progress and result events as they arrive.

```python
for event in client.search_stream(smiles="c1ccc(NC(=O)c2ccccc2)cc1", max_results=50):
    if event["type"] == "progress":
        print(event["stage"], event["pct"])
    elif event["type"] == "result":
        print(event["compound"]["smiles"], event["compound"]["similarity"])
```

## 4. Place a synthesis order (`order`)

Submit selected compounds for synthesis quoting; you get a tracked order id and
status.

```python
order = client.order(
    smiles_list=[
        "Cc1ccc2c(C(=O)Nc3ncc(F)c(Cl)c3F)cc(-c3cccnc3)nc2c1C",
        "Fc1cc(CNc2ccnc(Oc3ccccc3)c2)c(F)cc1Br",
    ],
    email="you@example.com",
)
print(order["id"], order["status"])
```

## Conventions & error handling

- Auth: API key on the client (see authentication/onepot-authentication.yml).
- Streaming: server-sent events (`type` = `progress` | `result`).
- Bounding: use `max_results` rather than cursor pagination.
- No documented idempotency-key or rate-limit headers; see
  conventions/onepot-conventions.yml.
