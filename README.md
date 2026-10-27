# OpenVenue — an open standard for vendor-agnostic routing of tokenized markets

**Status:** `v0.0.1-draft` · working draft, not yet published · started 2026-10-27
**Name:** OpenVenue · **What it is:** an open standard for describing markets,
books, and quotes across heterogeneous tokenized venues · **The document:**
[spec/openvenue.md](spec/openvenue.md)

OpenVenue standardizes the data contract between venues that list tokenized
instruments and the routing systems that aggregate them. A venue adapter
translates one vendor's identifiers, wire format, and fee rules into canonical
OpenVenue documents; a routing core consumes those documents and never sees a
vendor-specific payload.

> **Open spec, closed code.** OpenVenue standardizes the *data contract* —
> schemas and vocabularies. It says nothing about how a venue produces its
> book or how a router chooses between venues. Anyone can implement it
> privately and keep their engine proprietary.

## The one idea

- **Canonical market** — every instrument, on every venue, resolves to a
  namespaced canonical id owned by the standard. Vendor identifiers travel on
  the adapter's mapping record, never as the canonical key.
- **Equivalence layer** — the same instrument listed on two venues maps to one
  canonical id, so depth and prices compare like-for-like.
- **All-in quote** — a quote is the effective price after venue fees, network
  cost, and the protocol fee, not the headline book price.
- **Venue boundary** — a venue is an opaque string key. Vendor identifiers,
  payload shapes, and fee rules live in the adapter and nowhere else.

## Layout

```
openvenue/  (this repository)
  README.md            ← you are here
  spec/
    openvenue.md       ← the normative reference specification
  LICENSE              ← CC BY 4.0
```

## Relationship to OpenBook

OpenVenue is part of the OpenBook Data Standards family. OpenBook standardizes
sportsbook and gambling data; OpenVenue standardizes the market and quote
contract for tokenized venues. They share the same design values — neutral
namespaced ids, open contracts, closed implementations.

The specification is licensed **CC BY 4.0**; implementations may be fully
proprietary.
