# Changelog

All notable changes to `com.intyga:intyga-verify` are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [SemVer](https://semver.org/).

## [Unreleased]

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
- Cryptography is the JDK's own (SHA-256, P-256 ECDSA); the only runtime dependency is Jackson,
  because Java has no standard-library JSON.
