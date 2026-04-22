- Feature Name: `zero-mail-at-skill-v0`
- Start Date: 2026-04-22
- RFC PR: (will be populated)
- Tracking Issue: (will be populated)

## Summary

Zero-Mail@Skill v0.1-draft is the wire protocol for the Zero ecosystem. It
binds three primitives into one coherent surface:

1. A **signed envelope** with eleven fixed fields and a canonical JSON
   serialisation.
2. A **content-addressed skill manifest** where `skill-id = sha256(canonical
   manifest)`.
3. **Ed25519 principals**: an agent's identity is its public key, and the
   envelope's signature over a fixed-shape input anchors every message to
   that key.

On top of those primitives the protocol defines a three-surface invariant
(admin inbox / public `/two` widget / zerobox hook must read the same rows),
a capability model with budget and TTL, and an email-shell addressing
convention (`{agent_id}@mail.zero.ski`, `@skill` local-part) so the whole
thing can tunnel through existing SMTP/IMAP infrastructure without losing
authenticity.

This RFC retroactively codifies the protocol already shipped in
[`tokencube/zero-protocol`](https://github.com/tokencube/zero-protocol). The
normative text lives in that repository's `SPEC.md`; this RFC is the
rationale, the argument for the design, and the baseline every subsequent
RFC builds against.

## Motivation

The Zero ecosystem needs agents to talk to each other — across trust
boundaries, across hosts, across time — with three properties that existing
messaging standards do not give us together:

- **Authentic.** Every message is signed by a specific principal, and
  verification requires no DNS lookup, no CA, no revocation list. If I
  archive a message today I can still verify it in ten years from the raw
  bytes and a known-pubkey-set.
- **Content-addressed.** Every skill (the executable unit an agent invokes)
  has a stable, verifiable identity derived from its manifest. Two agents in
  different time zones that claim to run `skill-id=abc…` are provably
  running the same code, byte-for-byte.
- **Auditable by default.** The envelope carries enough metadata
  (`message_id`, `in_reply_to`, `thread_id`, `from`, `to`, `cmd`,
  `timestamp`) that a replay over the trace log reproduces the conversation
  graph. There is no separate "metadata store" that can drift from the
  message stream.

The existing primitives we already use internally (a `trace` table that is
the system's event log, a zerobox container that executes skills, a worker
that routes email and HTTP into the same dispatcher) imply this protocol;
they do not specify it. When three engineers (or three agents) write three
implementations without a shared spec they produce three subtly different
wire formats. This RFC draws the line.

The goal is that any conforming implementation — Rust kernel, TypeScript
worker, Python zerobox runner, future Go rewrite — accepts and produces
byte-identical envelopes and computes byte-identical skill IDs. Golden
vectors in `tokencube/zero-protocol` enforce that.

## Guide-level explanation

Think of a Zero agent as a Unix process that lives inside an email
mailbox. Its identity is an Ed25519 keypair; its address is
`{agent_id}@mail.zero.ski`, where `agent_id` is a stable handle derived
from the public key. Its "commands" are `@skill` invocations: the recipient
local-part or an attachment selects a content-addressed skill, which is the
only thing that may actually run.

A Zero message is an **envelope**, a JSON object with eleven fields fixed
in order and shape. Implementations serialise it with a canonical JSON
rule: sorted keys, no insignificant whitespace, RFC 8259-compliant strings.
Canonical serialisation is what you sign over; canonical serialisation is
what you hash. Two implementations that agree on the rule agree on every
downstream derivation.

The envelope carries a **signature** — Ed25519 over a fixed-shape signing
input built from the envelope's own fields. Absent optional fields are
represented as empty strings, not as omitted lines; this matters because
adding a new optional field must never retroactively invalidate old
signatures. Verification is: reconstruct the signing input, look up the
sender's public key (from a known-pubkey-set or from the content itself if
self-describing), and check. No DNS, no CA, no revocation.

Each envelope names a **skill**. A skill is a manifest (JSON describing
entry point, input schema, required capabilities, WASM binary hash if
applicable). The skill's identity is the SHA-256 of its canonical manifest.
When an agent says "please run `skill-id=a1b2c3…`" the verifier either has
that manifest or fetches it from a content-addressed store and checks the
hash. There is no "skill name" that can be repointed behind your back.

On top of envelope + skill the protocol defines **capabilities**. A
capability is itself a signed envelope-shaped object that says: "issuer
grants subject the right to invoke skill-id X with scope Y, from time A
to time B, bounded by budget Z USD, nonce N". No central revocation; short
TTL is the revocation mechanism. A capability is the DMARC analog for
skill invocations — policy about who is allowed to do what, expressed in
the same primitive as the messages it governs.

Finally, the protocol mandates a **three-surface invariant**: the admin
inbox page, the public `/two` MAIL widget, and the zerobox operator hook
must all read the same underlying rows — computed by a single function
`buildAgentInboxRows(traces, self_agent_id)` over a 24-hour rolling
window. Drift between the three surfaces is an alertable condition, not a
cosmetic bug. The invariant is how we force the protocol to be the single
source of truth instead of letting each UI reimplement it.

## Reference-level explanation

The full normative text is
[`tokencube/zero-protocol/SPEC.md`](https://github.com/tokencube/zero-protocol/blob/main/SPEC.md).
This RFC summarises the structural decisions.

### Envelope (SPEC §5-7)

The envelope has eleven fields, in canonical-JSON key order:

| Field | Purpose |
|---|---|
| `v` | Protocol version tag (`"zero-mail/0.1"` for v0.1-draft). |
| `message_id` | Globally unique ID, RFC 5322-compatible. |
| `in_reply_to` | Parent `message_id` or `""` for root. |
| `thread_id` | Conversation grouping; equals `message_id` for a root. |
| `ts` | RFC 3339 timestamp with `Z` suffix (UTC). |
| `from` | Sender address `{agent_id}@mail.zero.ski`. |
| `to` | Recipient address, may be a `@skill` local-part. |
| `cmd` | The skill invocation intent (string, schema-validated per skill). |
| `skill_id` | `sha256(canonical manifest)` of the skill being invoked. |
| `cap` | Optional capability token, or `""` if none. |
| `sig` | Ed25519 signature over the canonical signing input. |

The signing input is the envelope with `sig` set to the empty string,
canonicalised, then fed to Ed25519. Absent optional values are `""`, not
omitted keys — so signatures remain stable when future RFCs tighten
semantics of a field without adding new ones.

### Canonical JSON (SPEC §6)

- Keys sorted lexicographically by UTF-8 codepoint.
- No whitespace between tokens.
- Strings per RFC 8259, with UTF-8 encoding.
- Numbers in their shortest valid decimal representation.

The well-known `1e21` edge case (where JavaScript and Go disagree on
canonical form) is called out in SPEC §6.4; implementations MUST use the
shortest round-trip form and MUST include the golden vector for
`1e21` in their test suite.

### Skill manifest & skill-id (SPEC §11)

A skill manifest is a JSON object; `skill-id` is the SHA-256 of its
canonical serialisation. A manifest's fields include entry point,
input/output schemas, required capability scopes, and (if executable) the
hash of the binary. The binary is fetched separately from a
content-addressed store; the manifest is what grounds the skill's
identity.

### Capability model (SPEC §8)

A capability is a signed object with `issuer`, `subject`, `skill_id`,
`scope`, `not_before`, `not_after`, `usd_budget`, `nonce`, `sig`. There
is no revocation list. TTLs are short (typical: minutes to hours). Scope
is a compositional string (grammar in SPEC §8.3); two overlapping scopes
MUST be resolved by intersection, never union. If two capabilities authorise
the same skill invocation, the verifier picks the one with tighter scope,
not looser.

### Three-surface invariant (SPEC §3)

Exactly one function —
`buildAgentInboxRows(traces, self_agent_id) → Row[]` — is the source of
truth for what an agent's mailbox looks like. The admin page, the public
widget, and the operator hook each render that row set; none of them may
compute their own. Divergence between surfaces is a P1 alert.

### Corner cases

- **Clock skew.** Verifiers MUST accept a `ts` within ±5 minutes of their
  own wall clock (SPEC §7.5). A capability's `not_before`/`not_after` is
  checked strictly; the 5-minute tolerance applies only to envelope `ts`.
- **Empty string vs. missing.** `in_reply_to: ""` and omitting the key are
  NOT equivalent at the signing layer. The canonical form requires all
  eleven keys present; absent values are `""`. Implementations that omit
  keys will produce unverifiable signatures.
- **Unicode normalisation.** Addresses and `cmd` MUST be NFC-normalised
  before canonicalisation. Golden vectors include a combining-mark case.

## Drawbacks

- **Email shell is heavier than raw JSON-RPC.** Tunnelling through
  SMTP/IMAP means implementations carry MIME parsing, RFC 5322 header
  handling, and DKIM-shaped verification logic. A pure JSON-over-WebSocket
  transport would be simpler. The email shell was chosen because it gives
  us free mailbox semantics, human-readable archives, and compatibility
  with existing audit/e-discovery tooling — but the complexity cost is
  real.
- **Canonical JSON has the `1e21` edge case.** Agreeing on canonical form
  across JavaScript, Go, Rust, and Python is not free. We carry golden
  vectors in the protocol repo specifically for this reason, but it
  remains a sharp edge that new implementations can trip on.
- **Ed25519 has no post-quantum resistance.** A sufficiently capable
  quantum adversary breaks every signature ever produced under this
  protocol. A future RFC will define a Dilithium (or then-current
  PQ-signature) upgrade path with a dual-signature migration window. v0
  deliberately defers this.
- **No revocation list.** Short TTL is an acceptable revocation mechanism
  in a well-connected system but is worse than CRL/OCSP in offline or
  partitioned settings. Agents in a partition will honour capabilities
  longer than an online system would.
- **Three-surface invariant is operational, not cryptographic.** An
  implementation could pass all golden vectors and still silently diverge
  between its admin UI and its public widget. The invariant is enforced
  by alerting, not by the protocol itself. This is intentional (the
  protocol should not dictate UI) but it is a loose joint.
- **Retroactive RFC.** Ratifying a protocol that is already shipping is
  uncomfortable — the debate comes after the code. The alternative
  (writing the RFC first) was abandoned in practice; this RFC records
  reality. Future RFCs must front-run their implementations.

## Rationale and alternatives

### Why signed envelope + content-addressed skill

The combination, not either half, is what delivers the property we want:
*replayable authenticity*. A signed envelope with a mutable skill
reference gives you "this person said please run X" where X can change
meaning later. A content-addressed skill with no envelope signature gives
you "this code ran" with no proof of *who* asked. Both together give you
"this specific key asked for this specific code at this specific time",
and that is what an audit trail needs.

### Why email-style mailbox metaphor

Three reasons, in decreasing order of weight:

1. **Human-readable archives for free.** A protocol trace viewed as an
   `.mbox` is legible to operators who have never read the spec. This
   matters for incident response and regulatory review.
2. **Existing infrastructure.** SMTP delivery, IMAP retrieval, DKIM-shaped
   verification logic — we inherit an enormous body of battle-tested
   code. We do not need to invent new transport.
3. **Mental-model match.** Agents have inboxes. Users have inboxes. The
   metaphor is the territory; no translation layer.

The metaphor is also tested as an A/B hypothesis (see project memory
`project_mailbox_as_metaphor_not_primitive.md`): if it turns out to be
wrong, we expect to find out by observing what queries humans actually
send. The protocol is designed so that dropping the email shell — just
carrying envelopes over WebSocket — is a local substitution, not a
rewrite.

### Alternatives considered

- **gRPC + mTLS.** Rejected. mTLS binds identity to DNS and CAs; our core
  invariant is that authenticity survives DNS outages and CA breaches. A
  Zero message must be verifiable from bytes alone, decades later. mTLS
  does not provide that.
- **CBOR envelopes.** Rejected. CBOR is marginally more efficient on the
  wire but its canonical form has its own (different) edge cases, and
  CBOR is not human-readable for audit. JSON's legibility is a feature
  for a protocol whose whole point is auditability.
- **JOSE / JWT.** Rejected. JWTs ship "ambient authority" — possession of
  the token is authority — with no built-in primitive for chained intent
  ("I authorise A to authorise B on my behalf, for skill X, bounded by
  Y"). Our capability model needs that chain; layering it on top of JWT
  means fighting JWT's defaults.
- **ActivityPub.** Prior art rather than alternative. ActivityPub
  federates actor-to-actor messages, but without a signing discipline
  strong enough for financial actions. We borrow the "actor is a URL"
  shape and replace the trust model with Ed25519 anchoring.
- **Nostr.** Prior art. Nostr is the closest existing system — Ed25519
  pubkey identity, signed events — and we unapologetically borrow its
  identity model. The difference is that Nostr events are social posts
  and Zero envelopes are command invocations with capability budgets;
  the two protocols solve overlapping but distinct problems.
- **Bluesky AT Protocol.** Prior art. Signed records with content
  addressing for repos. Closer to Zero than ActivityPub. The main
  divergence is that AT Proto centralises handle resolution on DNS; Zero
  deliberately does not.
- **X.509 / PKI.** Counter-example. We reject the premise that a CA can
  bind identity to a key; that is exactly the property we want to avoid
  needing.

### Why content-addressing with SHA-256 specifically

SHA-256 is boring. It is ubiquitous, audited, and has no known practical
collisions. For a v0 protocol, pinning a specific hash is better than
writing "any cryptographically secure hash". A future RFC will define a
migration path to whatever post-SHA-256 primitive the field settles on
(likely SHA-3 or BLAKE3); the migration path will look like the PQ
signature migration — dual-hash envelopes during a transition window.

### Impact of not doing this

The alternative to ratifying v0 is that each implementation's current
behaviour is de facto the spec, and every subsequent implementation has
to reverse-engineer the others. That is how protocols fragment into
mutually incompatible dialects (see: any JavaScript module format from
2010-2015). Writing this down is cheap; not writing it down is very
expensive in one to two years.

## Prior art

- **SPKI/SDSI** (Rivest, Lampson et al., 1996–1999). The capability model
  in SPEC §8 is essentially SPKI with email-shaped syntax. The scope
  composition rule (intersect, never union) is directly from SPKI.
- **Nostr** (Fiatjaf, 2021). Ed25519 pubkey identity, signed events,
  content-addressable relays. We borrow the identity primitive almost
  unchanged.
- **Bluesky AT Protocol** (2023). Signed records, content-addressed repos,
  DID-based handle resolution. Strong influence on how we think about
  mutable pointers to immutable records.
- **X.509 / Web PKI**. Deliberate counter-example. The list of CA
  breaches, misissuances, and trust-store politics is the argument for
  not building on DNS + CA.
- **ActivityPub** (W3C, 2018). Federation model we partially follow
  (actor URIs, inboxes) without its loose trust posture.
- **DKIM / DMARC** (RFC 6376, RFC 7489). The capability model in SPEC §8
  is the DMARC analog for skill invocations — policy expressed as signed
  data, verifiable by anyone holding the public key.
- **Git content addressing**. The skill manifest SHA-256 is Git's object
  model applied to executable capability descriptors.
- **Ken Thompson, "Reflections on Trusting Trust" (1984).** The reason
  skills are content-addressed: you can only trust a binary you can hash.

## Unresolved questions

These are the open items the author (with owner) expects to resolve
before stabilisation — through follow-up RFCs or during v0→v1:

1. **Handle-alias resolution.** v0 addresses an agent by
   `{agent_id}@mail.zero.ski` where `agent_id` is derived from the
   pubkey. A human-memorable alias (`alice@mail.zero.ski` → pubkey) is
   desirable but the resolution mechanism (DNS TXT? well-known endpoint?
   in-band attestation?) is not specified. Will need its own RFC.
2. **Capability composition when scopes overlap.** SPEC §8.3 says
   "intersect, never union" but the precise grammar of a scope and the
   intersection algorithm for non-trivial scopes (e.g. scope-with-budget
   ∩ scope-with-TTL) is not fully pinned. Implementations currently
   agree on the common cases; the corner cases need formalising.
3. **Skill manifest schema evolution.** If we add a field to the manifest
   in v1, every old skill-id changes. The migration strategy — dual-hash
   period? versioned manifests? — is deferred.
4. **Post-quantum signature migration.** Dilithium or SPHINCS+ is the
   likely path, with a dual-signature overlap window. Needs a dedicated
   RFC.
5. **Binary skill distribution.** The manifest references a binary by
   hash; where the binary is fetched from (content-addressed store?
   swarm? fallback HTTPS?) is implementation-defined in v0. A standard
   discovery mechanism is a future RFC.
6. **Clock skew tolerance.** ±5 minutes is a guess. Whether it should be
   configurable per-surface, and how offline operation interacts with
   strict capability `not_after` checking, needs more operational data.
7. **Three-surface invariant as a conformance test.** Currently the
   invariant is enforced by alerting in a single deployment. Making it
   a first-class conformance test for independent implementations
   requires deciding what "the same 24h rolling window" means when
   clocks are not synchronised.

## Future possibilities

- **Capability-attenuated delegation.** A natural extension: "I authorise
  A to re-authorise a strict subset of my grant to B". SPKI has this;
  v0 capabilities are single-hop; multi-hop delegation with provable
  attenuation is a likely v1 feature.
- **Content-addressed skill marketplace.** Because skill-id is a hash, a
  global skill registry is trivially mirrorable and forkable. A future
  RFC may standardise a discovery protocol so that "run
  `skill-id=abc…`" can find a binary without the sender pre-distributing
  it.
- **Post-quantum signatures.** Dilithium or whatever wins the transition
  window. Dual-signed envelopes during migration.
- **Zero-knowledge capabilities.** "Prove you have a capability in scope
  X without revealing the capability itself" — likely needed for
  privacy-preserving audit. Research-grade.
- **Homogeneous transport options.** WebSocket, QUIC, or libp2p as
  transports alongside SMTP/IMAP. Envelope is transport-independent by
  design; future RFCs can standardise alternatives.
- **Federated trust anchoring.** Well-known-key endpoints served over
  HTTPS from a domain the sender controls, as a fallback when
  known-pubkey-set distribution is too heavy. Orthogonal to the core
  protocol.
- **Budget telemetry.** A capability's `usd_budget` implies a metering
  mechanism; standardising how spend is reported back to the issuer
  (and by whom) is a future RFC.

These are notes for future RFCs, not promises. An accepted v0 does not
commit the ecosystem to any of them.
