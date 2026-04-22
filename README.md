# Zero Ecosystem RFCs

Request-for-Comments process for the [Zero](https://zero.ski) ecosystem — the
protocol, kernel, and cross-project conventions maintained by
[`tokencube`](https://github.com/tokencube).

Modeled on the [Rust RFC process](https://github.com/rust-lang/rfcs), adapted
to the Zero stack.

## Why this process exists

The Zero ecosystem spans several repositories:

- [`tokencube/zero-protocol`](https://github.com/tokencube/zero-protocol) — the Zero-Mail@Skill protocol specification, golden vectors, and JSON schemas.
- [`tokencube/zeroski`](https://github.com/tokencube/zeroski) — the Rust kernel (signed envelope verifier, content-addressed skill loader, Ed25519 anchoring).
- Downstream implementations (private and public) that rely on the protocol for interop.

Small, ad-hoc PRs across these repos tend to fragment the design: one repo
grows a field the others don't understand, a capability scope drifts, two
implementations disagree on canonical JSON edge cases. This repository exists
so that **substantive changes get argued through in writing, once, in a single
place, before they land**.

Accepted RFCs become protocol canon. Implementations follow.

## When you need an RFC

File an RFC for any of the following:

- A **breaking change** to the envelope wire format, signature input shape, or canonical JSON rule.
- A **new capability string** (the `scope` field) or a change to how scopes compose.
- A **new `MUST`-clause** in the spec — anything that constrains conforming implementations.
- Governance changes: maintainer list, code of conduct, licensing of the protocol text, dual-license policy of the Rust kernel.
- Cross-project conventions: signing algorithm, handle-alias resolution, skill-id canonicalisation, capability-budget semantics.
- New surfaces the three-surface invariant must cover (beyond `/admin/inbox`, `/two` MAIL widget, and zerobox hook).

If you are unsure, file an RFC — a closed RFC with a clear rejection reason is
still valuable history.

## When you do **not** need an RFC

- Bug fixes in a single implementation.
- Documentation improvements, typo fixes, clearer wording that does not change normative behaviour.
- Additive examples, golden vectors, conformance fixtures.
- Internal refactors that are invisible at the protocol level.
- Performance improvements that preserve all observable behaviour.

These go straight to the relevant implementation repo as a normal PR.

## The process

1. **Fork** this repository.
2. **Copy** `0000-template.md` to `rfcs/0000-<your-slug>.md`. Keep the
   `0000` prefix — the maintainer assigns the real number on merge.
3. **Fill in** every section of the template. Leave `RFC PR:` and
   `Tracking Issue:` blank; they are populated by the maintainer.
4. **Open a PR** against this repo titled `RFC: <short title>`.
5. **Discuss**. Reviewers leave comments on the PR. The author updates the RFC
   in place. Expect several rounds; RFCs are not rushed.
6. **Final comment period (FCP)**. When discussion has settled, the maintainer
   announces a 10-day FCP on the PR. Any substantive objection raised during
   FCP pauses the clock until resolved.
7. **Decision**:
   - **Accepted** — the PR is merged, the file is renamed to the assigned
     number (e.g. `rfcs/0007-foo-bar.md`), a tracking issue is opened, and
     the `RFC PR` / `Tracking Issue` fields are backfilled.
   - **Rejected** — the PR is closed with a written reason. The rejection
     is itself part of the public record; the PR branch and discussion
     remain accessible.
   - **Postponed** — the PR is closed with a `postponed` label. The author
     (or anyone) may reopen a new RFC on the same topic later; the
     postponed PR is linked as prior art.

## Merge criteria

An RFC may be merged when **both** of the following hold:

- The BDFL ([@zhanjun](https://github.com/xczhanjun)) approves.
- No maintainer has an unresolved substantive objection.

"Substantive" means an objection about the design, scope, or compatibility
implications — not style or wording, which the author handles inline.

Maintainer list lives in [`.github/CODEOWNERS`](.github/CODEOWNERS). Changes
to that list are themselves RFC-governed.

## After acceptance

An accepted RFC is a **historical document**. It describes the design as it
was argued and approved. Amendments to accepted RFCs require a new RFC that
supersedes the old one; do not edit merged RFCs in place except for:

- Backfilling `RFC PR` / `Tracking Issue` links at merge time.
- Typo / formatting / dead-link fixes (open an issue, not a PR).
- Adding a "Superseded by RFC-NNNN" line at the top when replaced.

Accepted RFCs get a tracking issue on the implementing repository (typically
[`tokencube/zero-protocol`](https://github.com/tokencube/zero-protocol) or
[`tokencube/zeroski`](https://github.com/tokencube/zeroski)). Implementation
PRs reference the RFC number in their description.

## Bootstrap RFC

The first RFC — [`rfcs/0001-zero-mail-at-skill.md`](rfcs/0001-zero-mail-at-skill.md) —
retroactively documents the Zero-Mail@Skill v0.1-draft protocol that is
already shipped in `tokencube/zero-protocol`. It is the baseline every
subsequent RFC builds against.

## Licensing

RFC text in this repository is dual-licensed under
[Apache-2.0](LICENSE-APACHE) and [MIT](LICENSE-MIT), matching the Rust
ecosystem convention for technical documents. By submitting a PR you agree
your contribution is licensed under the same terms.

Commits must carry a `Signed-off-by` trailer (`git commit -s`) asserting the
[Developer Certificate of Origin](https://developercertificate.org/).

## Contact

- Conduct concerns: `conduct@zero.ski`
- General: open an issue on this repository.
