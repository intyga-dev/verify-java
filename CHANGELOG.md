# Changelog

All notable changes to `com.intyga:intyga-verify` are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [SemVer](https://semver.org/).

## [Unreleased]

- **Wire format: the DIV Intent Payload gained a REQUIRED `evidence` field, and it must be `null`.**
  `div-intent-verification` and `div-offline-intent` now carry `"evidence":null` in the signed bytes
  (DIV §4.3.4); `div-delegation`, `div-agent-authority` and `div-platform-intent` deliberately do
  not. `null` is signed and load-bearing, exactly as `requester.attestation`'s null is: it is the
  payload's explicit statement that the authorization was not conditioned on any external fact.
  Verification refuses a payload whose `evidence` key is absent, and refuses any non-`null` value
  rather than treating it as unconditioned — the same fail-closed-on-unknown rule as the
  `signerClass` registry, and checked before Local Payload Reconstruction so an unsupported payload
  shape does not surface as a parameter mismatch. Absent and `null` are distinguished explicitly;
  collapsing them would make the check a no-op. All golden vectors were regenerated.

- Align cross-language receipt and audit verification: platform receipts, agent-authority seals,
  self-certifying DID trust, single/multi-event bundles, embedded ES256 signatures, tenant sequence
  checks, checkpoint continuity, anchor quorum and Rekor. Shared executable fixtures cover valid
  artifacts and refusals; no wire format changes.
- Refuse unknown witness signature algorithms. Require identity-bound trust when the signed
  `requesterCannotApprove` rule is set; key-only trust cannot enforce requester identity.
- **`verifyRootsChain` range-checks the FIRST entry.** The non-integer and self-inverted seq-range
  checks were gated on having a predecessor, so the first entry was never range-checked — and a
  single-entry `roots.jsonl` (a new tenant, or the first day after a truncation) is exactly where
  the first entry is the only one. Such a file verified clean here while Go and Rust refused it.
  Only the overlap check is relational now. New `single-entry-inverted-seq-range` and
  `single-entry-non-integer-seq-range` parity vectors pin it in all five ports.
- Report a partially chained roots file as `roots file mixes chained and unchained entries` with
  `verifiedCount` 0, decided before the loop — it used to fall into the loop and credit the entries
  preceding the splice, under a reason string no other port used.

- Recheck a verified delegation's expiry when it is used, under the approval call's `asOf`, clock
  skew and explicit `allowExpired` forensic override.
- **Refuse a forward-dated offline proof or delegation (DIV §5a.3 rule 3, §5a.6 step 1).** The
  window caps bounded a proof's WIDTH but never its POSITION, so a quorum-signed proof dated years
  ahead with a compliant 60-minute (or 72-hour) window verified today and kept verifying until that
  date. The check is unconditional — the audit/`allowExpired` override re-examines a proof that was
  valid and has lapsed, and does not reach one dated in the future.
- **Refuse a signed `requirement.requiredApprovals` below 1 (DIV §4.3.2).** §5 step 7's "at least
  `requiredApprovals`" is satisfied vacuously by 0, so the minimum is now enforced explicitly
  instead of by an undocumented floor.
- **Range-check ECDSA `r` and `s` before handing a signature to the platform provider.** On JDK
  15-18 before 17.0.3 (CVE-2022-21449) an all-zero signature verifies under any key, which in this
  library is a total verification bypass. `verify-go` and `verify-rust` reject zero scalars inside
  their crypto libraries; Java delegated the whole check to the JDK, and this port targets release
  17.
- **Fail closed on a missing `expected.Target`, `expected.Nonce` or `expected.Approvers`.**
  `verifyApprovalReceipt` coerced an absent target and nonce to `""` and dereferenced a null trust
  anchor; all three now return the refusal the TypeScript reference does (DIV Target Isolation, §5
  step 10, Invariant 3). `verifyDelegation` gained the same anchor guard, which it previously
  null-checked in one place and dereferenced in another.
- **Return a refusal instead of throwing on malformed attacker-supplied JSON.** A null entry inside
  `signatures`, a null inside `allowedAaguids`/`delegatedTo`, and a null `bounds`/proof step in
  `Ledger.verifyMerkleProof` each raised a `NullPointerException` out of a method documented to
  return a result. Canonicalization is unchanged: a null set element sorts as the text `null`,
  exactly as the TypeScript reference's default comparator does.
- **Canonicalize `BigDecimal` params** — the type a mapper configured with
  `USE_BIG_DECIMAL_FOR_FLOATS` produces — accepting it only when the shortest form of the
  corresponding double reads back as the same number, so Java can never emit digits another port
  would not.

## [1.0.0]

Initial public release.

- Offline DIV receipt verification: canonical payload recomputation (RFC 8785 JCS), ES256 and
  WebAuthn signature checking, distinct-identity quorum counting, four-eyes, expiry with clock skew,
  and the fail-closed `signerClass` rule.
- Offline approvals (DIV §5a) behind an explicit opt-in, with the 60-minute window enforced at
  verification; `verifyDelegation` for §5a.5 delegations, with the 72-hour window and the refusal to
  let a delegation authorize anything by itself.
- DEWP Core Profile ledger verification: domain-separated hashing, duplicate-last Merkle trees,
  position-bounded inclusion proofs (§11.1), the `trust.intyga.audit.v1` preimage, and single-anchor
  ES256 signatures over the raw 32-byte digest.
- Canonicalization is pinned by the shared golden vectors in `packages/mcp-schemas/vectors/`, so
  Java is a sixth mirror of the canonical payload builder and moves with the other five.
- Doubles are formatted as the shortest round-tripping decimal rather than via `Double.toString`,
  which emits extra digits before JDK 19 — a receipt verified on Java 17 and one verified on Java 21
  must recompute identical bytes.
- Cryptography is the JDK's own (SHA-256, P-256 ECDSA); the only declared runtime dependency is
  `jackson-databind` (which brings `jackson-core` and `jackson-annotations` transitively), because
  Java has no standard-library JSON.
