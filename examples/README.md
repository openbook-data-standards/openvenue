# OpenVenue worked examples

One synthetic two-venue routing run end to end. `venue-a` and `venue-b` are
opaque venue keys; their native market ids appear only inside
`market-equivalence.json`, never in canonical documents.

| File | Document | Shows |
| --- | --- | --- |
| `canonical-market.json` | CanonicalMarket | The standard's registry entry for the NFL total 47.5 |
| `market-equivalence.json` | MarketEquivalence | Each venue's own id for that canonical market |
| `equivalence-match-report.json` | EquivalenceMatchReport | Coverage: 1 matched market plus a venue-b listing that resolved to nothing |
| `orderbook.venue-a.json` / `orderbook.venue-b.json` | OrderBook | Per-venue probability books at one instant |
| `fee-profile.json` | FeeProfile | 0.1% taker fee, no network fee |
| `quote-request.json` | QuoteRequest | Stateless request: books and fee profiles supplied with the request |
| `venue-quote.json` | VenueQuote | Result of walking venue-b's depth, fees included |
| `quote-decision.json` | QuoteDecision | The routing core's answer: best route, per-venue breakdown |

Walk in this order: canonical market → equivalence (how venue ids map) →
books → request → per-venue quote → decision. The decision's
`all_in_price` is the only number a consumer should compare across venues.

## Validate an example

Schemas are draft 2020-12. From the repository root:

```bash
pip install 'jsonschema>=4.18' referencing
python3 - <<'PY'
import json, pathlib
from jsonschema import Draft202012Validator
from referencing import Registry, Resource

root = pathlib.Path('.')
schemas = {p.name: json.loads(p.read_text())
           for p in root.glob('schema/*.schema.json')}
registry = Registry().with_resources(
    [(s['$id'], Resource.from_contents(s)) for s in schemas.values()])

for doc in sorted(root.glob('examples/*.json')):
    stem = doc.name.split('.')[0].replace('_', '-')
    schema = schemas.get(f'{stem}.schema.json')
    if schema is None:
        print(f'{doc}: no matching schema — checked by name convention')
        continue
    Draft202012Validator(schema, registry=registry).validate(
        json.loads(doc.read_text()))
    print(f'{doc}: OK')
PY
```

`orderbook.*.json` files match `orderbook.schema.json` by prefix, so validate
those two explicitly if you want a strict per-file check.

## What an adapter must add

- Its opaque venue key, its own market ids inside `venue_market_ids`, and its
  fee profile — nothing vendor-shaped anywhere else.
- Probability prices strictly inside (0, 1) and epoch-millisecond timestamps;
  validation fails closed, never clamps.
- `resolutionVariance` on the canonical market when settlement rules differ
  between venues that would otherwise match.
