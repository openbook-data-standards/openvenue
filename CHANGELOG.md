# Changelog

## Unreleased

Contract depth folded in from Proposal 0001 (all eight items decided):

- Normative precedence stated in the spec: prose wins over schemas/examples.
- Book ordering required (bids desc, asks asc, best first); consumers must
  re-sort regardless.
- Crossed/locked books are legal documents; a core refuses the quote rather
  than assuming both sides fill.
- Equivalence is all-or-nothing; line differences are different canonical
  markets, settlement differences carry `resolutionVariance`.
- Canonical id normalization fixed: lowercase, explicit `param` sign,
  bytewise equality.
- `split_plan` units defined (USD per venue, sums to the requested budget).
- `side` bounded by market shape; inexpressible outcomes are reported
  unmatched, never approximated.
- Staleness declared caller-side; adapters preserve venue timestamps.

## v0.0.2-draft — 2026-10-28

Data model aligned to the running implementation it was extracted from.

- OrderBook now carries separate `bids` and `asks`; prices are probability
  prices in (0, 1).
- QuoteRequest gains `feeProfiles` as a per-venue map and the `Outcome` side
  vocabulary (home, away, draw, over, under, yes, no).
- New models: CanonicalMarket, VenueQuote, QuoteDecision, AllInQuote,
  MarketEquivalence, EquivalenceMatchReport.
- Canonical id adopts the real segment structure
  `<sport>:<league>:<season>:<fixture>:<market_type>[:<param>]`; the
  `openvenue:` namespace replaces the implementation's `kaz:` prefix.
- Signer model added: WalletKind, Capability, Enforcement, SessionKeyMode.
- Schemas and examples regenerated from the model.

## v0.0.1-draft — 2026-10-27

Working draft. Canonical id format, equivalence layer rules, the data model
(OrderBookLevel, OrderBook, FeeProfile, QuoteRequest, Quote), the venue
boundary, and non-custodial settlement posture. Not yet published.
