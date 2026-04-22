# Accepted RFCs

This directory contains accepted RFCs in the order they were merged.

| Number | Title | Status | Tracking |
|-------:|-------|-------|----------|
| [0001](0001-zero-mail-at-skill.md) | Zero-Mail@Skill v0.1-draft | Accepted (bootstrap) | [`tokencube/zero-protocol`](https://github.com/tokencube/zero-protocol) |

## How numbers are assigned

Authors open PRs with the placeholder filename `rfcs/0000-<slug>.md`. On
acceptance the maintainer renames the file to the next free number and
backfills `RFC PR` and `Tracking Issue` in the frontmatter. The PR number
and the RFC number are not related.

## Status conventions

- **Accepted** — merged; normative for the implementing repository.
- **Superseded** — merged once, replaced by a later RFC; the top of the
  file carries a `Superseded by RFC-NNNN` line. The text stays in place
  as historical record.
- **Withdrawn** (rare) — merged, later found invalid; the top of the file
  carries a `Withdrawn: <reason>` line and a pointer to the decision PR.

Rejected and postponed RFCs are **not** in this directory; they live as
closed PRs on the repository, searchable by label.
