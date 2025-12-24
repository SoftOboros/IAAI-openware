# Socratic Rebuttal: AI API Provenance Extension (AIPE) — draft-iaai-aipe-00

> **Intent**: A question-driven rebuttal meant to *stress-test* the AIPE proposal, not to dismiss the provenance goal.  
> If AIPE is going to be a standards-track profile, the burden is to prove it is (a) **necessary**, (b) **minimally sufficient**, and (c) **operationally adoptable**.

---

## 0. Shared premises (before we argue)

1. **Does provenance prove “truth,” or merely “origin + integrity”?**  
   If we agree it’s the latter, why does the draft sometimes read like provenance is a general-purpose trust solution rather than a narrowly-scoped security primitive?

2. **Is “enforced provenance” compatible with high availability?**  
   If the system fails closed on registry fetches, network partitions, or suite mismatches, are we prepared for provenance to become a new single point of failure?

3. **What is the smallest standard that still stops the attacks we care about?**  
   If we can solve the real threat with 20% of the machinery, why standardize the other 80% as “Level 5 MUSTs”?

---

## 1. Scope: Why does one extension bundle three distinct standards problems?

AIPE attempts to standardize **(A) input document verification**, **(B) output model attestation**, and **(C) quality/coherence feedback** in one profile.

- **Question:** If input provenance is valuable on its own, why is it normatively tied to output attestation and Quality Indicators?  
- **Question:** If output attestation is valuable on its own, why must it be implemented via the same registry and cryptographic suite machinery as document envelopes?
- **Question:** If Quality Indicators are explicitly “receiver-computed,” why are they a protocol requirement rather than an application-level metric?

**Follow-on:** If providers adopt only the input side (the most defensible security win), does the standard still “work,” or does “Level 5” become non-viable by design?

---

## 2. Threat model: Which adversary does AIPE actually defeat?

1. **Data poisoning is the headline threat — but where does poisoning happen?**  
   If malicious chunks enter *before* the API call (e.g., inside the client’s retrieval layer), what incremental safety does API-boundary verification provide?

2. **What is the “attacker” in the model substitution story?**  
   - If the provider is honest-but-curious, attestation is useful.  
   - If the provider is malicious, signing an attestation proves only that the provider (or whoever holds the key) lied consistently.

3. **Do we assume the key holder is always the model executor?**  
   AIPE says attestations MUST be generated at response generation time, not at a gateway. But how would anyone detect violations without a hardware-backed execution proof?

**Follow-on:** Are we trying to prove **model identity**, **model execution**, or merely **provider accountability**?

---

## 3. “Model attestation”: Is this a proof of execution, or a signed receipt?

AIPE’s `model_attestation` binds a hash of the request and some identifiers into a VRF proof.

- **Question:** What does a VRF proof add here over a normal signature scheme?  
  If the goal is only “provider signed a receipt,” why not use widely-deployed signature containers (JWS/COSE/CMS) and avoid novel verification paths?

- **Question:** If we actually want *proof the model executed*, where is the root of trust?  
  Without a TEE/remote attestation claim (or equivalent), isn’t this indistinguishable from a provider-generated statement?

- **Question:** How stable is `canonical_request_body` across clients?  
  If two compliant clients serialize JSON differently, do they compute different inputs and therefore fail verification?

**Follow-on:** If attestation cannot prove execution, should the standard scope it explicitly as **provider-signed accountability** and stop implying stronger guarantees?

---

## 4. Registries: Why invent DPKR instead of reusing existing key discovery?

AIPE requires resolving a domain registry (`/.well-known/dpkr.json`) for issuers and providers.

- **Question:** What problem does DPKR solve that JWKS (`/.well-known/jwks.json`), WebPKI certificates, or DID methods cannot?  
- **Question:** Are we standardizing a *new PKI* under a new name? If so, how do we prevent the same failures and governance issues that PKI already struggles with?
- **Question:** What happens in offline / air-gapped / regulated environments?  
  If registry fetching is mandatory and fail-closed, do we make “verification” impossible exactly where it matters most?

**Follow-on:** Why not allow the envelope to carry **sufficient key material** (or a verifiable certificate chain) to validate offline, with registries used only for rotation/revocation?

---

## 5. Availability vs. security: Does “fail closed” create an easy DoS?

AIPE’s security posture leans toward strict rejection when verification fails or registries are unreachable.

- **Question:** If an attacker can make registry fetches fail (routing, DNS, TLS interception, rate limiting), can they deny service to all “Level 5” requests?
- **Question:** Is a global cache TTL a safe default across all contexts?  
  Short TTLs increase availability risk; long TTLs increase revocation latency.

- **Question:** Do we need a standard “grace mode”?  
  Example: allow cached keys beyond TTL with explicit “verification_stale=true” signaling, rather than hard failure.

**Follow-on:** If the standard doesn’t define graceful degradation, will every implementer invent their own incompatible version anyway?

---

## 6. SSRF and supply-chain risks: Should endpoints be required to fetch `ref_uri`?

AIPE says that for referenced payloads, “The endpoint MUST fetch and verify the referenced ciphertext.”

- **Question:** Is this a safe MUST?  
  If the client can supply `ref_uri`, then the provider becomes a high-value SSRF proxy by default.

- **Question:** What are the normative constraints on `ref_uri`?  
  Are private IP ranges forbidden? Are redirects forbidden? Is there a max size / content-type allowlist / checksum-first policy?

- **Question:** Why is the fetch responsibility on the AI provider at all?  
  Why not require the **client** (or a dedicated retrieval service) to resolve and supply the bytes, so the AI endpoint doesn’t become a network fetcher?

**Follow-on:** If the “MUST fetch” remains, the security considerations section should treat SSRF as a first-class threat with concrete mitigations.

---

## 7. Document formats: What does “decrypt and inject plaintext” mean for PDFs?

AIPE’s processing requirements say: decrypt payload and inject plaintext into model context.

- **Question:** If `ptype` is `application/pdf`, what is “plaintext”?  
  Does Level 5 require a standardized PDF-to-text extraction pipeline? OCR? Embedded text only?

- **Question:** If extraction is provider-specific, how can clients reason about what the model actually saw?  
  Does this break auditability (the very thing provenance is supposed to improve)?

**Follow-on:** Should the standard limit verified document payloads (for context injection) to **textual canonical forms** (e.g., `text/plain`, `text/markdown`), while allowing binary formats only as *attachments* not *context*?

---

## 8. Key compromise and revocation: Are the semantics strong enough?

AIPE defines errors like `key-compromised` and speaks about registry refresh and TTL.

- **Question:** What is the revocation model?  
  If a key is compromised, how fast can *all verifiers* learn this? What happens to already-issued envelopes?

- **Question:** If envelopes remain valid after compromise (because proofs verify), what policy stops replay of poisoned-but-valid historic chunks?

- **Question:** Does the standard need **issuer policy** embedded in envelopes (expiry, intended audience, allowed uses), so that “verification” is more than “signature checks out”?

**Follow-on:** Without a clear revocation + freshness story, “verified” may become a false sense of safety.

---

## 9. Mixed verified/unverified context: Is the spec internally consistent?

AIPE introduces:
- top-level `provenance.require_verification`
- context-level `require_verification`
- `reject_unverified` that can “silently exclude” documents

- **Question:** Which field is authoritative if they disagree?  
- **Question:** If `reject_unverified=false`, can a request still be “Level 5 compliant”?  
- **Question:** If silent exclusion occurs, how do we prevent *quiet failure* where the model answers without the intended verified context?

**Follow-on:** If “enforced provenance” is the point, silent exclusion should likely be prohibited (or at least must be loudly signaled in `provenance_summary` and stream events).

---

## 10. Quality Indicators: Are we standardizing the wrong layer?

AIPE’s `quality_indicators` includes dimensions like relevance, grounding_coverage, citation_accuracy, etc.

- **Question:** Who computes these values — provider, client, or both?  
  If client-computed, why are they in the provider response schema?  
  If provider-computed, why should clients trust them (and how do we prevent gaming)?

- **Question:** Are these *metrics* or *normative semantics*?  
  If two providers define “grounding_coverage” differently, does the field create false comparability?

- **Question:** Why not treat QI as a separate, optional telemetry channel?  
  Forcing it into the core response risks politicizing the standard and slowing adoption.

**Follow-on:** Would AIPE be stronger if it standardized **how to report evidence** (citations bound to verified envelopes), and left “quality scoring” outside the protocol?

---

## 11. Privacy: Is issuer/model metadata leakage acceptable by default?

AIPE itself acknowledges:
- issuer domain exposure in `provenance_summary`
- model version inference via `model_attestation`

- **Question:** If provenance reveals sources, does that enable competitive intelligence or legal exposure?  
  Should redaction be a MUST in some compliance profiles?

- **Question:** If model version exposure is sensitive, how can we preserve auditability without leaking operational details?  
  Do we need a “coarse attestation” mode standardized up front?

**Follow-on:** If privacy is optional, will most implementers ignore it — and then discover they cannot deploy in regulated settings?

---

## 12. Algorithm agility and the “10‑year test”: Is PQC first-class or an afterthought?

AIPE already references a PQC suite identifier in an example error, but the core story is still classical.

- **Question:** Is the registry + envelope design flexible enough to support hybrid post-quantum transitions without breaking verification?  
- **Question:** Should the spec mandate *algorithm agility* patterns now (multi-proof, hybrid signatures, suite negotiation), rather than retrofitting later?

**Follow-on:** If provenance is meant to outlive model versions, shouldn’t “quantum migration” be a primary design axis?

---

## 13. A constructive alternative: What if AIPE were split into three independent drafts?

1. **AIPE-Docs (Verified Context Documents)**  
   - Standardize envelope format, issuer discovery, verified citations.  
   - Keep it narrowly about defeating data poisoning and enabling audit.

2. **AIPE-Receipts (Provider Accountability Receipts)**  
   - Standardize an output “receipt” that can be signed with widely-deployed mechanisms.  
   - Avoid claiming proof-of-execution unless backed by TEEs.

3. **AIPE-QI (Optional Quality Telemetry)**  
   - Standardize a schema for optional quality reports that are explicitly non-authoritative and non-comparable.

**Question:** Would splitting increase adoption by letting implementers pick the smallest module they can ship?

---

## 14. Minimal edits that would materially improve the current draft

If the goal is to keep AIPE as one profile, consider:

- **Define canonicalization** (or define the `input` hash in a way that avoids canonical JSON problems).
- **Make `ref_uri` fetching optional** (or strictly constrained), and define SSRF mitigations normatively.
- **Clarify the trust claim** of `model_attestation` (receipt vs execution).
- **Resolve redundancy** between top-level and context-level verification flags.
- **Standardize privacy modes** (issuer redaction, coarse model attestation).
- **Move QI to OPTIONAL** (or to an extension registry), so the security core can ship first.

---

## Closing question

If we imagine an implementer reading AIPE and asking:

> “What is the one thing I must build to measurably reduce real-world risk without turning my API into a cryptography product?”

What is AIPE’s answer — and is that answer small enough to be adopted widely?

