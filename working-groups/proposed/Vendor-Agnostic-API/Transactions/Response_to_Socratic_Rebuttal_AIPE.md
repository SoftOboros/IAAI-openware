# Response to Socratic Rebuttal: AI API Provenance Extension (AIPE)

**Document:** draft-iaai-aipe-00 Response Statement  
**Date:** 2025-06-23  
**Status:** Working Group Discussion

---

## Preamble

The Socratic Rebuttal raises substantive concerns that strengthen this specification. We accept several critiques as grounds for revision, defend certain design choices as intentional, and propose concrete amendments where the rebuttal identifies genuine ambiguity or overreach.

The organizing principle of this response: **AIPE should be the smallest standard that stops the attacks we care about, while remaining extensible for environments that require more.**

---

## 0. On Shared Premises

### 0.1 "Does provenance prove truth, or merely origin + integrity?"

**Accepted.** Provenance proves origin and integrity—nothing more. Where the draft implies broader trust guarantees, this is editorial overreach that will be corrected.

**Amendment:** Section 1.1 will be revised to explicitly state:

> "Provenance verification establishes that content originated from a claimed source and has not been modified in transit. It does NOT establish that the content is accurate, truthful, complete, or fit for any particular purpose. Provenance is a necessary but insufficient condition for trust."

### 0.2 "Is enforced provenance compatible with high availability?"

**Acknowledged as a real tension.** The rebuttal correctly identifies that fail-closed semantics create availability risk. However, the alternative—fail-open—defeats the security goal entirely.

**Position:** AIPE defines the *security-maximal* posture. Deployments requiring higher availability MAY implement grace modes, but these MUST be explicitly signaled (not silent). See Section 5 response for proposed `verification_stale` signaling.

### 0.3 "What is the smallest standard that still stops the attacks?"

**This is the right question.** The response below identifies which AIPE requirements are essential versus which should be demoted to OPTIONAL or split into separate profiles.

---

## 1. On Scope: Why Bundle Three Problems?

### The Critique

AIPE bundles (A) input document verification, (B) output model attestation, and (C) quality feedback into one profile. The rebuttal asks whether these are separable.

### Response

**They are separable, and AIPE should acknowledge this explicitly.**

The three components address different threat actors:

| Component | Threat Addressed | Adoption Barrier |
|-----------|------------------|------------------|
| Input verification | Data poisoning by external attackers | Low (client-side tooling) |
| Output attestation | Provider accountability / model substitution | Medium (provider key management) |
| Quality indicators | Coherence feedback for self-tuning | Low (optional telemetry) |

**Amendment:** AIPE will define three compliance sub-profiles:

- **Level 5a (Verified Context):** Input verification only. Providers accept SPEVR envelopes and enforce verification. No output attestation required.
- **Level 5b (Attested Output):** Output attestation only. Providers sign responses. No input verification required.
- **Level 5 (Full Provenance):** Both 5a and 5b, plus optional QI.

A provider implementing only 5a still delivers the primary security win (defeating data poisoning) without the key management overhead of attestation.

**Quality Indicators are demoted to OPTIONAL** across all sub-profiles. They remain specified for interoperability but are not normative for compliance.

---

## 2. On Threat Model: Which Adversary Does AIPE Defeat?

### 2.1 "Where does poisoning happen?"

**The rebuttal is correct that poisoning often occurs before the API call.** AIPE's API-boundary verification is a defense-in-depth layer, not a complete solution.

However, API-boundary verification provides value in three scenarios:

1. **Multi-tenant RAG services** where the retrieval layer is shared infrastructure and individual clients cannot trust what others have indexed.
2. **Federated pipelines** where documents traverse multiple systems before reaching the model.
3. **Audit and compliance** where the API boundary is the natural enforcement point for logging verified sources.

**Amendment:** Section 1 will clarify:

> "AIPE is most effective when combined with provenance enforcement at ingestion time. API-boundary verification serves as a defense-in-depth checkpoint and audit surface, not as a substitute for secure ingestion pipelines."

### 2.2 "What is the attacker in model substitution?"

**Accepted: AIPE cannot detect a fully malicious provider.** If the provider controls the signing key and lies about which model executed, the attestation proves only consistent lying.

**Position:** AIPE's model attestation provides **provider accountability**, not **execution proof**. It creates a signed, non-repudiable record that the provider claims a specific model generated the response. This is valuable for:

- Contractual enforcement ("You billed me for GPT-4 but your attestation says GPT-3.5")
- Audit trails in regulated industries
- Dispute resolution

**Amendment:** Section 5.1.1 will be revised:

> "Model attestation provides **provider accountability**: a cryptographic receipt binding the provider's claimed model identity to the response. It does NOT constitute proof of execution. Proof of execution requires hardware-backed attestation (e.g., TEE/SGX) which is outside AIPE's scope. Future extensions MAY define TEE attestation bindings."

### 2.3 "Do we assume the key holder is always the model executor?"

**No.** AIPE assumes only that the key holder is accountable for the claim. The "MUST be generated at response generation time" language is a policy statement that providers SHOULD NOT delegate signing to untrusted gateways—but AIPE cannot enforce this cryptographically without TEEs.

**Amendment:** Revise to:

> "Attestations SHOULD be generated as close to model execution as architecturally feasible. Providers MUST NOT delegate attestation signing to components that do not have access to actual model execution context."

---

## 3. On Model Attestation: Proof vs. Receipt

### 3.1 "What does a VRF proof add over a normal signature?"

**Fair question.** The VRF was inherited from SPET's enforced provenance model where key derivation gates decryption. For model attestation—which is a signed receipt, not an access-control gate—a standard signature suffices.

**Amendment:** AIPE will support two attestation modes:

1. **`mode: signature`** — Standard ECDSA/EdDSA/RSA signature over the attestation payload. Simpler, uses existing JOSE/COSE tooling.
2. **`mode: vrf_vdkd`** — VRF-based binding for environments that want consistency with SPET envelope semantics.

The signature mode becomes the RECOMMENDED default for attestation. VRF mode is retained for deployments that prefer unified cryptographic patterns.

### 3.2 "How stable is canonical_request_body across clients?"

**This is a real problem.** JSON canonicalization is notoriously inconsistent.

**Amendment:** AIPE will mandate **RFC 8785 (JSON Canonicalization Scheme)** for computing request hashes:

> "The `input` hash MUST be computed over the RFC 8785 canonical form of the request body. Implementations MUST NOT use ad-hoc JSON serialization."

Alternatively, AIPE could define the `input` as a hash of explicitly enumerated fields (model, messages hash, timestamp) rather than the full body, reducing canonicalization surface.

---

## 4. On Registries: Why DPKR Instead of Existing Discovery?

### The Critique

DPKR duplicates functionality available in JWKS, WebPKI, or DID methods.

### Response

**Partially accepted.** DPKR was designed for the specific semantics of provenance key lifecycle (active/retired/revoked, compromise timestamps, suite restrictions). However, the rebuttal is correct that mandating a new registry format creates adoption friction.

**Amendment:** AIPE will define **registry profiles** that allow existing discovery mechanisms:

| Profile | Discovery Mechanism | Use Case |
|---------|---------------------|----------|
| `dpkr` | `/.well-known/dpkr.json` | Full DPKR semantics (key lifecycle, compromise handling) |
| `jwks` | `/.well-known/jwks.json` | Simpler deployments using existing OIDC infrastructure |
| `x509` | TLS certificate chain | Enterprise PKI integration |
| `did` | DID document resolution | Decentralized identity ecosystems |

The envelope's `registry_hint` field will accept a profile identifier. Verifiers resolve keys according to the indicated profile.

**For the `jwks` profile**, key status (active/revoked) is inferred from presence/absence in the current JWKS. Compromise timestamps are not available; deployments requiring this MUST use `dpkr`.

### 4.1 "What about offline/air-gapped environments?"

**Accepted.** Mandatory registry fetching breaks air-gapped deployments.

**Amendment:** AIPE will support **embedded key material**:

```json
{
  "type": "spevr_envelope",
  "envelope": { ... },
  "embedded_key": {
    "jwk": { ... },
    "certificate_chain": ["<base64>", "<base64>"]
  }
}
```

When `embedded_key` is present, verifiers MAY use it directly, with registry fetching used only for revocation checks (which can be skipped in air-gapped mode with explicit policy acknowledgment).

---

## 5. On Availability: Does Fail-Closed Create DoS?

### The Critique

If attackers can disrupt registry fetches, they can deny service to all Level 5 requests.

### Response

**Acknowledged.** This is an inherent tension in any fail-closed security system.

**Amendment:** AIPE will define a **grace mode** with explicit signaling:

```json
{
  "provenance": {
    "grace_mode": {
      "allow_stale_cache": true,
      "max_stale_seconds": 3600,
      "signal_staleness": true
    }
  }
}
```

When `allow_stale_cache` is enabled:

- Cached registries MAY be used beyond TTL up to `max_stale_seconds`
- If `signal_staleness` is true, responses include `"verification_stale": true` in `provenance_summary`
- Revocation checks are best-effort; newly revoked keys may not be detected

This allows deployments to choose their availability/security tradeoff explicitly rather than inventing incompatible workarounds.

**The default remains fail-closed.** Grace mode is opt-in.

---

## 6. On SSRF: Should Endpoints Fetch `ref_uri`?

### The Critique

Requiring providers to fetch `ref_uri` creates SSRF risk.

### Response

**Accepted. This is a serious concern that was under-specified.**

**Amendment:** AIPE will change the `ref_uri` requirement:

1. **Default behavior:** Providers SHOULD reject `ref_uri` payloads and require inline ciphertext. This is the safe default.

2. **Opt-in fetching:** Providers MAY support `ref_uri` fetching with the following normative constraints:
   - `ref_uri` MUST use `https://` scheme only
   - `ref_uri` MUST NOT resolve to private IP ranges (RFC 1918, RFC 4193, link-local)
   - `ref_uri` MUST NOT follow redirects (or follow only to same-origin)
   - Providers MUST enforce a maximum fetch size (RECOMMENDED: 10MB)
   - Providers MUST verify `ref_hash` before any processing
   - The `ref_uri` allowlist SHOULD be configurable per-deployment

3. **Alternative:** The RECOMMENDED architecture is **client-side resolution**: clients fetch the referenced payload, verify it locally, and submit inline. This removes SSRF risk entirely.

**Security Considerations** will be expanded with an SSRF-specific subsection.

---

## 7. On Document Formats: What Is "Plaintext" for PDFs?

### The Critique

If `ptype` is `application/pdf`, what does "inject plaintext into model context" mean?

### Response

**Accepted. This ambiguity undermines auditability.**

**Amendment:** AIPE will define two payload categories:

1. **Context-injectable payloads** (`text/plain`, `text/markdown`, `text/html`, `application/json`): These are injected directly into model context. The verifier confirms the decrypted bytes match the declared `ptype`.

2. **Attachment payloads** (all other types including `application/pdf`, `image/*`, etc.): These are verified but NOT automatically injected as text context. The model receives:
   - A reference to the verified attachment
   - Optionally, provider-extracted text (if the provider supports it), with explicit `extraction_method` metadata

```json
{
  "provenance_summary": {
    "verified_attachments": [
      {
        "document_id": "doc_3",
        "ptype": "application/pdf",
        "extraction": {
          "method": "pdf_text_layer",
          "extracted_text_hash": "<sha256>",
          "confidence": "high"
        }
      }
    ]
  }
}
```

This preserves auditability: clients know exactly what the model saw and how it was derived.

---

## 8. On Revocation: Are Semantics Strong Enough?

### The Critique

The revocation model is under-specified. What stops replay of compromised-but-valid historic chunks?

### Response

**Partially accepted.** AIPE inherits DPKR's revocation semantics but does not fully address replay.

**Amendment:** AIPE will add **envelope freshness constraints**:

1. **`max_envelope_age`** (request parameter): Verifiers MUST reject envelopes with `created_at` older than this threshold, regardless of key validity.

```json
{
  "provenance": {
    "max_envelope_age_seconds": 86400
  }
}
```

2. **Issuer-embedded policy** (envelope extension): Issuers MAY embed constraints:

```json
{
  "assertions": {
    "intended_audience": ["api.provider.com"],
    "valid_until": 1703433600,
    "single_use": false
  }
}
```

Verifiers SHOULD honor these constraints when present.

3. **Replay detection** remains application-layer. AIPE does not mandate deduplication databases, but the `envelope_id` field enables them.

---

## 9. On Mixed Verified/Unverified Context

### The Critique

Multiple verification flags create ambiguity. Silent exclusion enables quiet failure.

### Response

**Accepted. The redundancy is confusing and silent exclusion is dangerous.**

**Amendment:**

1. **Single authoritative flag:** Remove `context.require_verification`. The top-level `provenance.require_verification` is authoritative.

2. **Prohibit silent exclusion in strict mode:** When `require_verification: true`, the `reject_unverified` field is ignored—unverified documents always cause request failure.

3. **Explicit exclusion signaling:** When `require_verification: false` and `reject_unverified: true` (lenient mode), excluded documents MUST appear in `provenance_summary.excluded_documents` with reasons, AND a streaming event `document_excluded` MUST be emitted.

```json
{
  "provenance_summary": {
    "excluded_documents": [
      {
        "id": "doc_3",
        "reason": "no_envelope",
        "action": "excluded_from_context"
      }
    ]
  }
}
```

**The model MUST NOT silently answer without intended context.** If critical documents are excluded, the response should reflect this.

---

## 10. On Quality Indicators: Wrong Layer?

### The Critique

QI conflates metrics with semantics, may be gamed, and slows adoption.

### Response

**Largely accepted.** QI was included to complete the CDPN integration, but the rebuttal correctly identifies that it belongs in a separate layer.

**Amendment:**

1. **QI is moved to a separate extension:** `draft-iaai-aipe-qi-00` (Quality Indicator Extension)

2. **Core AIPE retains only citation binding:** The `citations` array with `envelope_id` and `verified` fields remains—this is evidence, not scoring.

3. **QI, if implemented, is explicitly non-authoritative:**

> "Quality Indicators are provider-computed heuristics. They are NOT comparable across providers, NOT normative for compliance, and SHOULD NOT be used as the sole basis for trust decisions. Clients requiring authoritative quality assessment MUST perform independent evaluation."

4. **Providers MAY omit QI entirely** and remain Level 5 compliant.

---

## 11. On Privacy: Metadata Leakage

### The Critique

Issuer domain and model version exposure may be unacceptable in some deployments.

### Response

**Accepted. Privacy modes should be standardized, not optional afterthoughts.**

**Amendment:** AIPE will define **privacy profiles**:

| Profile | Issuer Exposure | Model Exposure | Use Case |
|---------|-----------------|----------------|----------|
| `full` | Full domain | Full version | Open/audit environments |
| `redacted_issuer` | Hash only | Full version | Competitive sensitivity |
| `redacted_model` | Full domain | Family only | Operational security |
| `minimal` | Hash only | Family only | Maximum privacy |

Request parameter:

```json
{
  "provenance": {
    "privacy_profile": "redacted_issuer"
  }
}
```

In `redacted_issuer` mode, `provenance_summary` includes:

```json
{
  "verified_issuers": ["sha256:abc123..."],  // Hash, not domain
  "issuer_count": 2
}
```

Clients with the original envelopes can verify the hashes locally.

---

## 12. On PQC: First-Class or Afterthought?

### The Critique

The core story is classical. Quantum migration should be a primary design axis.

### Response

**Accepted.** AIPE should mandate algorithm agility patterns now.

**Amendment:**

1. **Suite negotiation:** Requests MAY include `preferred_suites` in priority order. Providers respond with the highest-priority mutually supported suite.

2. **Hybrid envelope support:** SPET already defines `mode: hybrid`. AIPE will explicitly support envelopes carrying both classical and PQC proofs, with verification succeeding if either proof validates (for migration) or both validate (for paranoid mode).

3. **Deprecation timeline:** AIPE will include an informative appendix recommending:
   - 2025-2027: Classical suites acceptable
   - 2027-2030: Hybrid suites RECOMMENDED
   - 2030+: PQC-only suites RECOMMENDED for new deployments

4. **Registry agility:** DPKR keys include `allowed_suites`, enabling per-key algorithm constraints during migration.

---

## 13. On Splitting AIPE

### The Proposal

Split into AIPE-Docs, AIPE-Receipts, AIPE-QI.

### Response

**Partially accepted.** The sub-profile approach (Section 1 amendment) achieves similar modularity without fragmenting the specification.

However, if the working group prefers separate documents:

- **draft-iaai-aipe-docs-00:** Verified context documents (Level 5a)
- **draft-iaai-aipe-receipts-00:** Provider accountability receipts (Level 5b)
- **draft-iaai-aipe-qi-00:** Quality telemetry (already proposed to split)

The parent **draft-iaai-aipe-00** becomes a profile document that combines them for "Full Level 5" compliance.

This allows implementers to cite exactly which component they support.

---

## 14. Summary of Amendments

| Section | Amendment |
|---------|-----------|
| 1.1 | Clarify provenance ≠ truth |
| 3.2 | Define Level 5a/5b/5 sub-profiles |
| 2 | Clarify API-boundary as defense-in-depth |
| 5.1 | Attestation is accountability, not execution proof |
| 5.1 | Support signature mode (not just VRF) for attestation |
| 5.1 | Mandate RFC 8785 canonicalization |
| 4 | Support multiple registry profiles (DPKR, JWKS, X.509, DID) |
| 4 | Support embedded key material for offline verification |
| New | Define grace mode with explicit staleness signaling |
| 6.2 | Default reject `ref_uri`; add SSRF mitigations if supported |
| 6 | Define context-injectable vs. attachment payload categories |
| 8 | Add `max_envelope_age` and issuer-embedded policy |
| 9 | Remove redundant flags; prohibit silent exclusion |
| 10 | Move QI to separate extension; demote to OPTIONAL |
| 11 | Define privacy profiles |
| 12 | Mandate algorithm agility; define hybrid support |

---

## 15. The Closing Question, Answered

> "What is the one thing I must build to measurably reduce real-world risk without turning my API into a cryptography product?"

**Answer: Level 5a (Verified Context Documents).**

An implementer who:

1. Accepts `spevr_envelope` content blocks
2. Resolves issuer keys via JWKS (existing infrastructure)
3. Verifies VRF proofs before context injection
4. Returns `provenance_summary` with verification results

...has implemented the primary security win (data poisoning resistance) with minimal cryptographic novelty. They use existing key discovery, existing signature verification libraries, and a well-defined envelope format.

Output attestation (5b) and quality indicators (QI extension) are valuable but separable. A provider can ship 5a in weeks, not months.

**This is AIPE's answer—and with the proposed amendments, it is small enough to be adopted widely.**

---

## Conclusion

The Socratic Rebuttal identified real weaknesses in draft-iaai-aipe-00:

- **Scope creep** bundling distinct concerns
- **Overreach** in trust claims (especially attestation)
- **Under-specification** of SSRF, canonicalization, revocation, privacy
- **Adoption friction** from mandatory registries and QI

The amendments above address these concerns while preserving AIPE's core value proposition: **enforced provenance at the AI API boundary.**

We invite the working group to review the proposed revisions and advise whether:

1. Amendments should be incorporated into draft-iaai-aipe-01, or
2. AIPE should be formally split into component specifications

Either path leads to a tighter, more adoptable standard.

---

**End of Response Statement**
