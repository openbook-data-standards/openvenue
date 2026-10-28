# Contributing to OpenVenue

OpenVenue is a draft standard. The fastest way to help is to implement an
adapter or a routing consumer against `spec/openvenue.md` and report what
broke.

## What contributions look like

- **Gap reports** — a real venue payload that cannot be expressed in the
  canonical model, or a market type with no vocabulary. Open an issue with a
  redacted sample document.
- **Ambiguity reports** — places where two reasonable implementers would read
  the spec differently. Quote the section; do not guess the fix.
- **Proposals** — normative changes land through a written proposal that names
  the section it amends and states who it affects. A pull request against the
  spec without a proposal will be redirected to open one.

## Ground rules

- Vendor identifiers never enter the spec or the schemas. Venue names in
  examples are opaque keys (`venue-a`, `venue-b`).
- The standard is the document. Schemas and examples are derived artifacts —
  if they disagree with `spec/openvenue.md`, the spec is right; fix the
  artifact.
- A version bump is the editor's call, recorded in `CHANGELOG.md`.
