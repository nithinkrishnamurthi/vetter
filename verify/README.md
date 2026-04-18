# vetter-verify

Offline verifier for vetter attestation chains.

**Status:** placeholder — shipping with Phase 2 of the moat roadmap.

When released, `vetter-verify` will:

- Consume a tenant's attestation log (`GET /tenants/:id/attestations`, NDJSON).
- Verify each entry's signature against vetter's published tenant public keys.
- Verify the RFC 3161 timestamp on each verdict.
- Verify the hash chain (`prev_attestation_hash` → current entry).
- Exit non-zero on any break, pointing at the specific entry that failed.

Apache-2.0. Shipped as a static Go binary so auditors can run it offline
without trusting the vetter cloud.

## Why this is open source

The point of a signed attestation log is that a third party can verify it
*without* the signer's cooperation. A closed-source verifier would defeat
that. The signing keys and attestation store stay in the closed vetter
cloud — the verifier is a thin library plus CLI that anyone can audit.
