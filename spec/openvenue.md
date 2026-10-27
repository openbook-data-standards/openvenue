# OpenVenue specification

- **Status:** `v0.0.1-draft` — working draft, not yet published
- **License:** CC BY 4.0
- **Scope:** the canonical market schema, the equivalence layer, and the quote
  contract exchanged between tokenized venues and vendor-agnostic routing
  systems.

OpenVenue defines the documents a venue adapter emits and a routing core
consumes. Everything a vendor knows about its own instruments — identifiers,
payload shapes, fee rules — is confined to the adapter. The core operates only
on canonical documents.

## 1. Terminology

| Term | Meaning |
| --- | --- |
| Venue | An opaque string key identifying a source of books (for example `venue-a`). Vendor identifiers never appear outside the adapter. |
| Instrument | A tradable unit listed by one or more venues: a spot asset, a perpetual contract, an event contract, a tokenized real-world asset. |
| Canonical id | The standard's namespaced identifier for one instrument and market. |
| OrderBook | A venue's levels for one canonical market at one instant. |
| FeeProfile | A venue's fee schedule: taker fee fraction and network fee in USD. |
| Quote | The all-in result of walking aggregated depth for a budget. |

## 2. Canonical identifiers

Every mapped instrument is assigned a deterministic canonical id:

```
openvenue:<class>:<series>:<event>:<market>[:<param>]
```

- `class` — the instrument class, for example `spot`, `perp`, `rwa`, `event`.
- `series` — the listing series or underlying namespace, for example `solana`,
  `tbill-usd`, `rates`.
- `event` — the instrument's canonical event or asset key.
- `market` — the market type, for example `spot`, `yes`, `no`.
- `param` — optional qualifier, for example a strike or expiry.

Example:

```
openvenue:rwa:solana:tbill-usd-2026:spot
openvenue:event:macro:fed-oct-decision-2026:yes
```

A venue's own identifier for the instrument is carried on the adapter's
mapping record. It is never used as, or mixed into, a canonical id.

## 3. Equivalence layer

The equivalence layer resolves disparate venue listings to the same canonical
id. Two listings are the same market only when they agree on:

1. **Underlying event or asset.** The instrument being priced is identical.
2. **Period and scope.** Both venues price the same duration and settlement
   scope.
3. **Resolution rule.** If settlement differs (for example void-on-tie versus
   split), the market carries a resolution-variance notice and is not silently
   joined.

Unmatched listings are reported per venue so a consumer can measure coverage
rather than guess.

## 4. Data model

### 4.1 OrderBookLevel

| Field | Type | Bounds |
| --- | --- | --- |
| `price` | decimal | finite, strictly greater than 0 |
| `size` | decimal | finite, >= 0 |

### 4.2 OrderBook

| Field | Type | Bounds |
| --- | --- | --- |
| `venue` | string | opaque venue key |
| `market` | string | canonical id |
| `levels` | OrderBookLevel[] | both sides as defined by the market type |
| `timestamp_ms` | integer | epoch milliseconds, no more than 60 s in the future |

### 4.3 FeeProfile

| Field | Type | Bounds |
| --- | --- | --- |
| `venue` | string | opaque venue key |
| `taker_fee_pct` | decimal | [0, 1] |
| `network_fee_usd` | decimal | finite, >= 0 |

### 4.4 QuoteRequest

| Field | Type | Bounds |
| --- | --- | --- |
| `market` | string | canonical id |
| `side` | enum | the market's side vocabulary |
| `budgetUsd` | decimal | (0, 50000] |
| `builderFeeBps` | decimal | [0, 10000] |
| `books` | OrderBook[] | one per venue, supplied with the request |
| `fees` | FeeProfile[] | one per venue, supplied with the request |

The request is stateless: the caller supplies every book and fee profile with
each request. Nothing about a venue lives in the core.

### 4.5 Quote

The result of walking the supplied books: the allocation across venues, the
blended all-in price, and the per-venue fills. The all-in price includes venue
fees, network cost, and the protocol fee — it is the only price a consumer
should compare.

## 5. The venue boundary

- A venue is an opaque key in every canonical document.
- Adapters translate vendor payloads into OpenVenue documents before the data
  reaches the core, and translate execution intents back into vendor payloads.
- Adapters hold no state and no business logic. Equivalence matching, fee
  translation, and id mapping are the adapter's whole job.
- The core never parses a vendor identifier.

## 6. Settlement posture

OpenVenue assumes non-custodial settlement: the end user signs every
transaction with their own keys and the router never holds funds. The standard
carries no custody vocabulary.
