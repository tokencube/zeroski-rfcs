# Contributing to Zero RFCs

Thank you for your interest in shaping the Zero ecosystem.

## The process

The full RFC process is described in [README.md](README.md). Short version:

1. Fork this repository.
2. Copy `0000-template.md` to `rfcs/0000-<slug>.md`.
3. Fill in every section except `RFC PR` and `Tracking Issue`.
4. Open a PR titled `RFC: <short title>`.
5. Participate in discussion on the PR; update the RFC in place.
6. On acceptance, the maintainer assigns a number, renames the file, and
   opens a tracking issue in the implementing repository.

## Before opening a PR

- Read [README.md](README.md) for the merge criteria and the definition of
  "substantive objection".
- Read [`rfcs/0001-zero-mail-at-skill.md`](rfcs/0001-zero-mail-at-skill.md)
  for the baseline the ecosystem is built on. Any RFC that conflicts with
  0001 must either supersede it with a dedicated migration plan or be
  rejected as out of scope.
- Check existing open and closed RFC PRs for prior discussion of your
  topic.

## Sign your commits

Every commit on a PR must carry a `Signed-off-by` trailer asserting the
[Developer Certificate of Origin](https://developercertificate.org/):

```
git commit -s -m "rfc: descriptive summary"
```

The DCO CI check will fail the PR otherwise.

## Typo and formatting fixes in accepted RFCs

**Accepted RFCs are historical documents.** They describe a design as it was
argued and approved. Do not open a PR to edit a merged RFC except for:

- Backfilling `RFC PR` / `Tracking Issue` links at merge time (maintainer task).
- Typo, formatting, or dead-link fixes — please open an **issue** rather
  than a PR. The maintainer will apply the fix directly or explain why
  not.
- Adding a "Superseded by RFC-NNNN" line at the top, only when a
  superseding RFC has itself been accepted.

Substantive changes to an accepted RFC's design require a new RFC that
supersedes it.

## Discussion etiquette

- Assume good faith. RFC reviewers disagree often; they rarely mean harm.
- Engage with the strongest version of the opposing argument.
- When you push back on a design choice, propose the concrete alternative
  you would accept, not just the one you reject.
- If a discussion is getting stuck, ask the maintainer to call a final
  comment period.

## Conduct

This project follows the [Contributor Covenant 2.1](CODE_OF_CONDUCT.md).
Report concerns to `conduct@zero.ski`.

## Licensing

By submitting a contribution (RFC text, PR discussion, issue comment) you
agree that it is licensed under the repository's dual Apache-2.0 + MIT
terms.
