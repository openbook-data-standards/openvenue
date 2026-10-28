# Changelog

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
