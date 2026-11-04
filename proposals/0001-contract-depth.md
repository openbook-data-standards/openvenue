# Proposal 0001 — Contract depth: precedence, ordering, and equivalence edge cases

**Status:** proposed — awaiting editor decision on each item.
**Amends:** `spec/openvenue.md` sections 2, 3, 4.2–4.7, and adds a new
precedence/conformance section.
**Affects:** venue-adapter implementers and routing-core consumers.

Each item below names a place two reasonable implementers would read the
`v0.0.2-draft` spec differently, then recommends one reading. An item marked
**decided** by the editor becomes normative text; anything else is not the
spec yet.

## P1 — Normative precedence is stated in CONTRIBUTING, not in the spec

Today "the spec is right; fix the artifact" lives in `CONTRIBUTING.md`, so a
reader of `spec/openvenue.md` alone cannot know which artifact wins.
OpenExchange states the opposite rule ("the schema wins"), so the convention
is not inferable from the family.

- **Proposed:** add a conformance note near the top of the spec — where prose
  and schema disagree, **prose wins**; schemas and examples are derived.
  Matches CONTRIBUTING.

## P2 — Book level ordering is unspecified (§4.2, §4.3)

`bids[]` and `asks[]` are arrays with no stated order. A walker that assumes
best-first and a venue that emits unsorted levels disagree silently, and the
result is a wrong `effective_price`, not an error.

- **Proposed:** venues emit `bids` sorted by `price` descending and `asks`
  sorted ascending (best level first). A consumer MUST NOT infer correctness
  from array order alone — it re-sorts or walks by best price. Emitting
  unsorted is a conformance failure, consuming without sorting is an
  implementation risk.

## P3 — Crossed and locked books have no stated posture

A book with best bid >= best ask is possible on real venues (crossed market,
cross-matching in flight, stale snapshot). The spec neither permits nor
rejects it.

- **Proposed:** crossed/locked books are legal documents. The adapter reports
  the book the venue reported; it does not un-cross or drop levels. A routing
  core treats a crossed book as a data anomaly, not a silent arb: it MAY
  refuse the venue's quote and MUST NOT execute on the assumption both sides
  fill. Rationale: fabricating an arb from a data artifact is worse than
  refusing.

## P4 — Equivalence is all-or-nothing; partial matches have no rule (§3)

Section 3 lists three agreement conditions but does not say what happens when
a venue listing matches on event and period but differs on line (`param`) or
resolution.

- **Proposed:** matching is all-or-nothing. A line difference is a different
  canonical market (it is part of the canonical id), not a `resolutionVariance`.
  A settlement-rule difference is `resolutionVariance` on the market and
  prevents the join. Anything else unmatched goes to `unmatched` in the
  `EquivalenceMatchReport` — never fuzzy-joined.

## P5 — Canonical id normalization is under-specified (§2)

The id grammar is shown but not the normalization rules: case, the signedness
of `param`, and fixture-slug date format are all free today.

- **Proposed:** canonical ids are lowercase; `param` carries an explicit sign
  for signed market types (`+47.5`, `-0.5`) matching the existing examples;
  the fixture date is `YYYY-MM-DD` in the fixture's canonical timezone as
  declared by the adapter's mapping record. Two ids equal only bytewise —
  no case-insensitive comparison.

## P6 — `split_plan` units are unstated (§4.7)

`split_plan` is `map<venue, decimal>` — allocation could mean USD budget,
contract count, or fraction of the order.

- **Proposed:** `split_plan` values are USD allocated to each venue; the sum
  equals `requested_budget_usd`. Absent key means zero allocation. (If the
  running implementation uses a different unit, the schema and this proposal
  change to match reality — the implementation wins, per the spec's
  provenance note.)

## P7 — `side` vocabulary vs market arity is uncoupled (§4.5)

`draw` in the side enum has no rule tying it to three-way markets; `yes`/`no`
vs `home`/`away`/`over`/`under` mixing is undefined.

- **Proposed:** `draw` is only valid where the canonical market's shape
  admits a draw outcome; `yes`/`no` belong to binary and event-contract
  markets; team/over-under sides belong to the corresponding sport market
  types. An adapter that cannot express a venue outcome in this vocabulary
  quarantines it (reports unmatched) rather than approximating.

## P8 — Staleness direction is half-specified (§4.3)

`timestamp_ms` rejects books more than 60 s in the *future*, but nothing is
said about old timestamps.

- **Proposed:** staleness is the caller's concern, not the schema's — the
  QuoteRequest is stateless and consumes whatever books the caller supplied.
  Adapters SHOULD preserve the venue's own timestamp rather than stamping
  receipt time. (No schema change; this fixes the asymmetry in prose.)

## What this proposal deliberately does not do

- No new wire fields, enums, or objects. Every item pins a reading of text
  that already exists or states an edge case the text is silent on.
- No router semantics — how a core weighs quotes beyond `all_in_price`
  remains out of scope ("open spec, closed code").
- `SessionKeyMode` and the §5 signer enums are named but unbounded; their
  value lists need a separate proposal once a second venue signer exists.
