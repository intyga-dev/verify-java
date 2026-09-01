# Changelog

All notable changes to `com.intyga:intyga-verify` are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [SemVer](https://semver.org/).

## [Unreleased]

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
