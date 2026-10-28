# OpenVenue specification

- **Status:** `v0.0.2-draft` — working draft, not yet published
- **License:** CC BY 4.0
- **Scope:** the canonical market schema, the equivalence layer, and the quote
  contract exchanged between tokenized venues and vendor-agnostic routing
  systems.
- **Provenance:** this draft is extracted from a running implementation. Every
  model below is in production use; nothing in the data model is aspirational.

OpenVenue defines the documents a venue adapter emits and a routing core
consumes. Everything a vendor knows about its own instruments — identifiers,
payload shapes, fee rules — is confined to the adapter. The core operates only
on canonical documents.

## 1. Terminology

| Term | Meaning |
| --- | --- |
| Venue | An opaque string key identifying a source of books (for example `venue-a`). Vendor identifiers never appear outside the adapter. |
| Instrument | A tradable unit listed by one or more venues: a spot asset, a perpetual contract, an event contract, a tokenized real-world asset. |
| Canonical market | The standard's registry entry for one instrument and market. |
| Canonical id | The standard's namespaced identifier for one canonical market. |
| OrderBook | A venue's bids and asks for one canonical market at one instant. |
| FeeProfile | A venue's fee schedule: taker fee fraction and network fee in USD. |
| VenueQuote | The per-venue result of walking that venue's book for a budget. |
| QuoteDecision | The routing core's answer: best route, per-venue quotes, optional split plan, expiry. |

## 2. Canonical identifiers

Every mapped market is assigned a deterministic canonical id:

```
openvenue:<sport>:<league>:<season>:<fixture>:<market_type>[:<param>]
```

- `sport` — the sport namespace, for example `football`, `basketball`,
  `baseball`, `hockey`, `soccer`.
- `league` — the listing league, for example `nfl`, `nba`, `mlb`, `nhl`, `mls`.
- `season` — the season key, for example `2026`.
- `fixture` — the canonical fixture slug, for example
  `<away-team>-at-<home-team>-<date>`.
- `market_type` — the market shape: `market:moneyline`, `market:spread`,
  `market:total`, or `market:binary`.
- `param` — optional qualifier, for example the line on a spread or total.

Example:

```
openvenue:football:nfl:2026:buf-at-kc-2026-11-02:market:total:+47.5
openvenue:football:nfl:2026:dal-at-phi-2026-11-09:market:moneyline
```

Non-sport instrument classes (for example `spot`, `perp`, `rwa`, `event`)
follow the same segment discipline; the class vocabulary is open and owned by
the standard, not by any venue.

A venue's own identifier for the market is carried on the adapter's mapping
record (`MarketEquivalence.venue_market_ids`). It is never used as, or mixed
into, a canonical id.

> **Namespace note.** The running implementation predates the `openvenue:`
> namespace and prefixes ids with `kaz:`. Migrating the prefix is a mechanical
> adapter change; the segment structure above is what both namespaces share.

## 3. Equivalence layer

The equivalence layer resolves disparate venue listings to the same canonical
market. Two listings are the same market only when they agree on:

1. **Underlying event.** The fixture or asset being priced is identical —
   canonical teams, scheduled date, and market shape.
2. **Period and scope.** Both venues price the same duration and settlement
   scope.
3. **Resolution rule.** If settlement differs (for example void-on-tie versus
   split), the market carries `resolutionVariance` and is not silently joined.

Unmatched venue listings are reported per venue (`EquivalenceMatchReport`) so
a consumer can measure coverage rather than guess.

## 4. Data model

Field names below are the wire names. Validation fails closed: out-of-bounds
input is rejected, never clamped.

### 4.1 CanonicalMarket

| Field | Type | Bounds |
| --- | --- | --- |
| `canonicalId` | string | canonical id format, section 2 |
| `sport` | enum | `football`, `basketball`, `baseball`, `hockey`, `soccer` |
| `league` | enum | `nfl`, `nba`, `mlb`, `nhl`, `mls` |
| `season` | string | season key |
| `homeTeam` | string | canonical team key |
| `awayTeam` | string | canonical team key |
| `scheduledStart` | string | ISO 8601 |
| `marketType` | enum | `market:moneyline`, `market:spread`, `market:total`, `market:binary` |
| `param` | decimal, optional | line or qualifier |
| `resolutionVariance` | string, optional | settlement difference notice |

### 4.2 OrderBookLevel

| Field | Type | Bounds |
| --- | --- | --- |
| `price` | decimal | probability price, strictly (0, 1); real venues quote 0.001 to 0.999, so no narrower clamp is applied |
| `size` | decimal | finite, >= 0; available contract count at the level |

### 4.3 OrderBook

| Field | Type | Bounds |
| --- | --- | --- |
| `venue` | string | opaque venue key |
| `bids` | OrderBookLevel[] | may be empty |
| `asks` | OrderBookLevel[] | may be empty |
| `timestamp_ms` | integer | epoch milliseconds; rejected if more than 60 s in the future |

### 4.4 FeeProfile

| Field | Type | Bounds |
| --- | --- | --- |
| `taker_fee_pct` | decimal | [0, 1], default 0 |
| `network_fee_usd` | decimal | finite, >= 0, default 0 |

### 4.5 QuoteRequest

| Field | Type | Bounds |
| --- | --- | --- |
| `canonicalMarketId` | string | canonical id |
| `side` | enum | `home`, `away`, `draw`, `over`, `under`, `yes`, `no` |
| `budgetUsd` | decimal | (0, 50000] |
| `builderFeeBps` | decimal | [0, 10000], default 0 |
| `books` | OrderBook[] | one per venue, supplied with the request |
| `feeProfiles` | map<venue, FeeProfile> | one per venue, supplied with the request |

The request is stateless: the caller supplies every book and fee profile with
each request. Nothing about a venue lives in the core.

### 4.6 VenueQuote

| Field | Type | Bounds |
| --- | --- | --- |
| `venue` | string | opaque venue key |
| `effective_price` | decimal | price after walking that venue's depth |
| `contracts_yielded` | decimal | contracts the budget buys at that venue |
| `total_spent_usd` | decimal | gross spend at that venue |
| `venue_fee_usd` | decimal | venue fees at that venue |
| `builder_fee_usd` | decimal | builder fee attributed to that venue |
| `all_in_price` | decimal | `total_spent_usd / contracts_yielded` |
| `depth_exhausted` | boolean | true when the venue's book could not fill the budget |

### 4.7 QuoteDecision

| Field | Type | Bounds |
| --- | --- | --- |
| `canonical_market_id` | string | canonical id |
| `side` | string | the requested outcome |
| `requested_budget_usd` | decimal | the requested budget |
| `best_route` | string | the venue key with the best all-in price |
| `all_in_price` | decimal | blended all-in price |
| `total_contracts_yielded` | decimal | contracts across the plan |
| `venue_quotes` | map<venue, VenueQuote> | per-venue breakdown |
| `split_plan` | map<venue, decimal>, optional | allocation when splitting beats best single venue |
| `expires_at` | datetime | quote validity deadline |

### 4.8 AllInQuote

The venue-adapter-facing all-in result: `executed_shares`,
`total_cost_usd`, `blended_price`, `base_cost_usd`, `fees_usd`,
`network_fee_usd`, and `max_slippage_price`. The all-in price includes venue
fees, network cost, and the protocol fee — it is the only price a consumer
should compare.

### 4.9 EquivalenceMatchReport

| Field | Type | Bounds |
| --- | --- | --- |
| `matchedCount` | integer | canonical markets matched on >= 1 venue |
| `totalCanonicalMarkets` | integer | canonical markets in the run |
| `canonicalMarkets` | CanonicalMarket[] | the matched set |
| `venueOnlyCounts` | map<venue, integer> | per-venue matched counts |
| `unmatched` | map<venue, string[]> | venue listings that resolved to no canonical market |

### 4.10 MarketEquivalence

| Field | Type | Bounds |
| --- | --- | --- |
| `canonical_id` | string | canonical id |
| `sport` | string | sport namespace |
| `league` | string | league namespace |
| `start_time` | datetime | fixture start |
| `venue_market_ids` | map<venue, string> | each venue's own market reference |

## 5. Signer model

Non-custodial execution needs a vendor-neutral description of who signs. The
standard carries four enumerations:

- `WalletKind` — the wallet form (`eoa`, `poly_proxy`, `poly_gnosis_safe`,
  `poly_deposit_wallet`, `solana`). Values are opaque to consumers.
- `Capability` — what a signer may do (`order_sign`, `batch_execute`,
  `withdraw`, `token_approve`, `permit`, `session_authorize`, `revoke`,
  `read_only`).
- `Enforcement` — who enforces a capability boundary (`on_chain`,
  `venue_enforced`, `provider_enforced`, `application_only`, `unknown`).
- `SessionKeyMode` — how a session key is scoped.

## 6. The venue boundary

- A venue is an opaque key in every canonical document.
- Adapters translate vendor payloads into OpenVenue documents before the data
  reaches the core, and translate execution intents back into vendor payloads.
- Adapters hold no state and no business logic. Equivalence matching, fee
  translation, and id mapping are the adapter's whole job.
- The core never parses a vendor identifier.

## 7. Settlement posture

OpenVenue assumes non-custodial settlement: the end user signs every
transaction with their own keys and the router never holds funds. The standard
carries no custody vocabulary.
