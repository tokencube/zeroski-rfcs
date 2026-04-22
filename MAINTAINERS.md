# Maintainers

## BDFL

- **Zhanjun** ([@xczhanjun](https://github.com/xczhanjun)) — Benevolent
  Dictator For Life. Final authority on RFC acceptance, rejection, and
  postponement. Timezone: UTC+8 (Hong Kong).

The BDFL is the tie-breaker, not the bottleneck. Review turnaround target is
within one week during the initial discussion phase and within 72 hours once
a final comment period has been announced.

## Reviewers

Reviewers may leave substantive objections that pause an RFC's merge until
resolved. There is no formal reviewer roster yet; the current de facto
reviewer pool is anyone who has landed an accepted RFC, plus the BDFL.

Adding a named reviewer list is itself an RFC-governed change — see
[`.github/CODEOWNERS`](.github/CODEOWNERS) and the process in
[`README.md`](README.md).

## Decision rule

An RFC may be merged when **both** hold:

- The BDFL approves.
- No reviewer has an unresolved substantive objection.

"Substantive" means an objection to design, scope, or compatibility — not
style or wording, which the author handles inline during review.

## Contact

- RFC discussion: open an issue or PR on this repository.
- Conduct concerns: `conduct@zero.ski`.
- Security-sensitive protocol questions: `security@zero.ski`.
