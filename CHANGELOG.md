# Changelog

All notable changes to the Zero RFC process itself — the governance
structure, template, and tooling — are recorded here. Accepted RFCs are
listed in [`rfcs/README.md`](rfcs/README.md), not here.

This file follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and [Semantic Versioning](https://semver.org/spec/v2.0.0.html) for the
process version, not for any protocol version (which lives in
`tokencube/zero-protocol`).

## [Unreleased]

## [0.1.0] - 2026-04-22 — Initial RFC process bootstrap

### Added

- RFC process documented in `README.md` — modeled on `rust-lang/rfcs`,
  adapted for the Zero ecosystem.
- `0000-template.md` with Summary / Motivation / Guide-level /
  Reference-level / Drawbacks / Rationale / Prior-art / Unresolved
  questions / Future possibilities sections.
- `CONTRIBUTING.md` describing PR flow and DCO sign-off requirement.
- `CODE_OF_CONDUCT.md` adopting Contributor Covenant 2.1 by reference;
  report address `conduct@zero.ski`.
- `MAINTAINERS.md` naming the BDFL and describing the decision rule.
- Dual licensing: code/tooling under MIT OR Apache-2.0; RFC text under
  CC-BY-SA-4.0 (`LICENSE-DOCS`).
- GitHub Actions DCO workflow (`.github/workflows/dco.yml`) that fails
  any PR with an un-signed commit.
- PR template and issue template for early-stage RFC proposals.
- `.github/CODEOWNERS` pointing all paths at the BDFL.

[Unreleased]: https://github.com/tokencube/zeroski-rfcs/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/tokencube/zeroski-rfcs/releases/tag/v0.1.0
