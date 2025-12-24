# Internet-Draft: AI API Provenance Extension (AIPE)

**Intended status:** Standards Track  
**Expires:** 2026-12-23  
**Version:** draft-iaai-aipe-01  
**Obsoletes:** draft-iaai-aipe-00

---

## Abstract

This document defines the **AI API Provenance Extension (AIPE)**: a specification layer that integrates provenance verification into standardized AI API interfaces. AIPE bridges content authenticity standards with the emerging "Dream API" unified AI interface.

This revision (draft-01) addresses fundamental architectural concerns raised during review of draft-00. Most significantly, this version introduces a **tiered provenance model** that distinguishes between:

- **Signature-based provenance** (authenticity without encryption) — suitable for public content, high-throughput RAG, and long-term archival
- **Encryption-based provenance** (enforced access control) — suitable for confidential content in controlled environments

The tiered model acknowledges that provenance and confidentiality are orthogonal concerns that SHOULD NOT be conflated except when the threat model explicitly requires it. This revision aligns with C2PA principles for public provenance while preserving encryption-based options for high-security deployments.

---

## Status of This Memo

This Internet-Draft is submitted in full conformance with the provisions of BCP 78 and BCP 79. Distribution of this memo is unlimited.

---

## Summary of Changes from draft-00

| Area | draft-00 | draft-01 |
|------|----------|----------|
| **Provenance model** | Encryption-only (SPEVR) | Tiered: Signature OR Encryption |
| **Compliance structure** | Single Level 5 | Levels 5.1, 5.2, 5.3, 5.4 |
| **Default mode** | Enforced encryption | Signature-based (C2PA-aligned) |
| **RAG optimization** | Not addressed | Async verification, metadata filtering |
| **Key management** | Per-envelope keys | Hierarchical with rotation support |
| **Archival concerns** | Not addressed | "Digital Vellum" mode |
| **Quality Indicators** | Normative | Moved to separate extension |
| **Registry profiles** | DPKR only | DPKR, JWKS, X.509, DID |
| **Model attestation** | VRF-only | Signature (default) or VRF |
| **Privacy profiles** | Not specified | Four standardized profiles |

---

## 1. Introduction

### 1.1 The Problem Revisited

Draft-00 correctly identified three gaps in current AI APIs:

1. **Data poisoning** — No verification of RAG document sources
2. **Model substitution** — No proof of which model generated a response  
3. **Coherence blindness** — No feedback mechanisms for semantic quality

However, draft-00 proposed a single solution—encryption-based enforced provenance—that conflates **authenticity** (origin verification) with **confidentiality** (access control). This conflation creates:

- **Performance crisis**: Decrypting k=500 RAG candidates adds 200-400ms latency
- **Operational burden**: Key rotation requires re-encrypting entire vector databases
- **Archival risk**: Key loss equals permanent data loss ("Cryptographic Shredding")
- **Scalability collapse**: Metadata filtering impossible on encrypted fields

### 1.2 The Revised Position

**Provenance and confidentiality are orthogonal concerns.**

- **Authenticity** answers: "Who created this, and has it been modified?"
- **Confidentiality** answers: "Who is permitted to read this?"

For most provenance use cases—public media, enterprise RAG, content credentials—**authenticity alone is sufficient**. Digital signatures provide tamper-evidence without hiding content behind encryption.

Encryption-based enforced provenance remains valuable for **specific high-security scenarios**:

- Confidential documents in multi-tenant RAG systems
- Cross-organizational data exchange with access control requirements
- Environments where unauthorized viewing constitutes a security breach

AIPE draft-01 provides **both options** through a tiered compliance model.

### 1.3 Design Principles

1. **Authenticity by default, confidentiality by choice**: Signature-based provenance is the baseline; encryption is opt-in.

2. **Fail-open for archives, fail-closed for security**: Public provenance degrades gracefully; confidential provenance fails safely.

3. **Operational sustainability**: Key management overhead scales with security requirements, not with data volume.

4. **Performance-aware**: Verification can be asynchronous; encryption latency is acknowledged and bounded.

5. **C2PA alignment**: Signature-based tiers interoperate with existing C2PA manifests.

---

## 2. Terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 2.1 Core Terms

* **Provenance Tier**: One of four compliance levels (5.1-5.4) with increasing security guarantees and operational overhead.
* **Signed Document**: A document with attached or detached cryptographic signature proving origin and integrity.
* **Encrypted Envelope**: A document wrapped in SPET encryption requiring key derivation for access.
* **Hard Binding**: Cryptographic hash linking signature to content bytes.
* **Soft Binding**: Perceptual hash or watermark surviving format transformations.

### 2.2 Verification Modes

* **Synchronous Verification**: Verification completes before content enters model context.
* **Asynchronous Verification**: Content enters context immediately; verification completes in parallel; results annotate the response.
* **Deferred Verification**: Verification metadata is recorded; actual verification occurs post-response or on audit.

---

## 3. Tiered Compliance Model

AIPE defines four provenance tiers as sub-levels of Dream API Level 5. Implementations MAY support any combination of tiers.

### 3.1 Tier Overview

| Tier | Name | Provenance Mechanism | Verification Mode | Key Management | Use Case |
|------|------|---------------------|-------------------|----------------|----------|
| **5.1** | Signed Context | Detached signatures (C2PA-compatible) | Async or Deferred | PKI certificates | Public RAG, media provenance |
| **5.2** | Attested Output | Response signatures | Sync | Provider PKI | Model accountability |
| **5.3** | Verified Context | Signed + sync verification | Sync | PKI + policy | Regulated industries |
| **5.4** | Enforced Provenance | Encrypted envelopes (SPEVR) | Sync (mandatory) | Per-document or hierarchical | Confidential RAG, cross-org exchange |

### 3.2 Tier 5.1: Signed Context (Recommended Default)

**Goal**: Provide tamper-evident provenance for RAG documents without encryption overhead.

**Mechanism**: Documents carry detached digital signatures or embedded C2PA manifests. The signature proves origin and integrity but does not restrict access.

**Verification**: 

- **Asynchronous** (default): Documents are injected into context immediately. Signature verification runs in parallel. Results appear in `provenance_summary`.
- **Deferred**: Signature metadata is logged. Verification occurs on audit request.

**Key Management**: Standard PKI. Issuer certificates are resolved via JWKS, X.509 chains, or DPKR registries. Certificate expiration does not destroy content access—only verification capability.

**Performance**: Near-zero latency impact. Signature verification is parallelizable and does not block retrieval.

**Archival Properties**: "Digital Vellum"—if certificates expire or algorithms deprecate, content remains readable. Only provenance verification degrades.

#### 3.2.1 Signed Document Format

```json
{
  "type": "signed_document",
  "content": {
    "type": "text",
    "text": "Q4 revenue increased 15% year-over-year..."
  },
  "signature": {
    "algorithm": "ES256",
    "value": "<base64url>",
    "certificate_chain": ["<base64>", "<base64>"],
    "signed_at": 1703347200
  },
  "hard_binding": {
    "algorithm": "SHA-256",
    "digest": "<base64url>"
  },
  "soft_binding": {
    "type": "c2pa_manifest",
    "manifest_uri": "self#jumbf=c2pa"
  }
}
```

#### 3.2.2 Request Schema (Tier 5.1)

```json
{
  "model": "claude-3-opus-20240229",
  "provenance": {
    "tier": "5.1",
    "verification_mode": "async",
    "allowed_issuers": ["research.acme.com"],
    "require_hard_binding": true
  },
  "context": {
    "documents": [
      {
        "id": "doc_1",
        "type": "signed_document",
        "content": { ... },
        "signature": { ... }
      }
    ]
  },
  "messages": [ ... ]
}
```

#### 3.2.3 Response Schema (Tier 5.1)

```json
{
  "id": "msg_123",
  "provenance_summary": {
    "tier": "5.1",
    "verification_mode": "async",
    "documents_submitted": 3,
    "documents_verified": 3,
    "documents_pending": 0,
    "verification_results": [
      {
        "document_id": "doc_1",
        "status": "verified",
        "issuer": "research.acme.com",
        "signed_at": 1703347200,
        "binding_intact": true
      }
    ]
  },
  "choices": [ ... ]
}
```

### 3.3 Tier 5.2: Attested Output

**Goal**: Provide cryptographic proof that a specific model generated the response.

**Mechanism**: Provider signs response metadata using a model-specific key. This creates a non-repudiable receipt.

**Trust Claim**: **Provider accountability**, not execution proof. The signature proves the provider claims this model generated the response. It does NOT prove the model actually executed (which would require TEE attestation).

**Key Management**: Providers publish model signing keys via DPKR or JWKS. Each model version SHOULD have a dedicated key.

#### 3.3.1 Attestation Format

```json
{
  "model_attestation": {
    "provider": "api.anthropic.com",
    "model_id": "claude-3-opus-20240229",
    "attestation_type": "accountability_receipt",
    "algorithm": "ES256",
    "payload": {
      "request_hash": "<sha256 of canonicalized request>",
      "response_hash": "<sha256 of response content>",
      "timestamp": 1703347200,
      "nonce": "<random>"
    },
    "signature": "<base64url>",
    "certificate_ref": "https://api.anthropic.com/.well-known/jwks.json#claude-3-opus-2024"
  }
}
```

#### 3.3.2 Canonicalization

To ensure consistent request hashing across clients, AIPE mandates **RFC 8785 (JSON Canonicalization Scheme)** for computing `request_hash`.

Alternatively, implementations MAY use a **field-enumerated hash**:

```
request_hash = SHA-256(
  model_id ||
  SHA-256(system_prompt) ||
  SHA-256(messages_json) ||
  SHA-256(documents_json) ||
  timestamp
)
```

This reduces sensitivity to JSON serialization differences.

### 3.4 Tier 5.3: Verified Context

**Goal**: Ensure documents are signature-verified BEFORE entering model context. Suitable for regulated industries where unverified content must not influence outputs.

**Mechanism**: Same as Tier 5.1, but verification is **synchronous** and **fail-closed**.

**Difference from 5.1**:

| Aspect | Tier 5.1 | Tier 5.3 |
|--------|----------|----------|
| Verification timing | Async/Deferred | Sync (blocking) |
| Unverified documents | Annotated in response | Request fails |
| Latency impact | Minimal | Moderate (parallelizable) |
| Use case | General RAG | Compliance-critical |

#### 3.4.1 Request Schema (Tier 5.3)

```json
{
  "provenance": {
    "tier": "5.3",
    "verification_mode": "sync",
    "fail_on_unverified": true,
    "max_verification_time_ms": 500
  }
}
```

If any document fails verification, the entire request fails with error type `provenance-verification-failed`.

#### 3.4.2 Timeout Handling

If verification exceeds `max_verification_time_ms`:

- **Strict mode** (`fail_on_timeout: true`): Request fails
- **Lenient mode** (`fail_on_timeout: false`): Document excluded, noted in `provenance_summary`

### 3.5 Tier 5.4: Enforced Provenance (Encryption-Based)

**Goal**: Ensure content CANNOT be read without successful provenance verification. The act of decryption IS the act of verification.

**Mechanism**: Documents are wrapped in SPET/SPEVR encrypted envelopes. Key derivation requires successful VRF proof verification against the issuer's public key.

**When to Use**:

- Confidential documents in multi-tenant systems
- Cross-organizational exchange with access control
- Environments where unauthorized viewing is a security breach

**When NOT to Use**:

- Public content (encryption adds overhead without security benefit)
- High-throughput RAG with latency SLAs < 500ms
- Long-term archival (key loss = data loss)

#### 3.5.1 Performance Acknowledgment

Tier 5.4 incurs significant overhead:

| Operation | Tier 5.1/5.3 | Tier 5.4 |
|-----------|--------------|----------|
| k=100 retrieval | ~50ms | ~150-250ms |
| k=500 retrieval | ~80ms | ~300-500ms |
| Reranking (100 docs) | ~100ms | ~200-400ms |

Implementations SHOULD:

- Limit `k` for Tier 5.4 retrievals
- Use hierarchical key management (document-group keys, not per-document)
- Consider hybrid approaches (Tier 5.1 for filtering, Tier 5.4 for final context)

#### 3.5.2 Key Management Strategies

To address rotation and operational overhead, Tier 5.4 supports:

**Strategy A: Per-Document Keys**
- Each document has a unique derived key
- Key rotation requires re-encryption
- Highest security, highest overhead

**Strategy B: Document-Group Keys (Recommended)**
- Documents share keys by group (e.g., by issuer, by date range)
- Rotation re-encrypts groups, not individual documents
- Balanced security/overhead

**Strategy C: Envelope Key Hierarchy**
- Documents encrypted with Data Encryption Keys (DEKs)
- DEKs encrypted with Key Encryption Keys (KEKs)
- KEK rotation does not require data re-encryption
- Lowest overhead, requires careful KEK protection

#### 3.5.3 SPEVR Envelope Format (Unchanged from draft-00)

```json
{
  "type": "spevr_envelope",
  "envelope": {
    "spe_version": "1.0",
    "id": "spe_abc123",
    "issuer_domain": "research.acme.com",
    "issuer_kid": "key-2024-q4",
    "suite_id": "SPE-VRF-EDWARDS25519-SHA512-TAI-AES256GCM",
    "created_at": 1703347200,
    "binding": {
      "mode": "vrf_vdkd",
      "input": "<base64url>",
      "proof": "<base64url>"
    },
    "payload": {
      "ciphertext": "<base64url>",
      "iv": "<base64url>",
      "tag": "<base64url>",
      "ptype": "text/plain"
    }
  }
}
```

#### 3.5.4 Archival Warning

Content encrypted under Tier 5.4 is subject to **Cryptographic Shredding**: if keys are lost, content is irrecoverable.

For long-term archival, implementers SHOULD:

- Maintain secure key escrow
- Plan for algorithm migration (PQC transition)
- Consider dual-format storage (encrypted for access control, signed copy for archive)

---

## 4. Mixed-Tier Requests

Requests MAY include documents at different tiers:

```json
{
  "provenance": {
    "default_tier": "5.1",
    "allow_mixed_tiers": true
  },
  "context": {
    "documents": [
      {
        "id": "doc_public",
        "type": "signed_document",
        "tier": "5.1",
        ...
      },
      {
        "id": "doc_confidential", 
        "type": "spevr_envelope",
        "tier": "5.4",
        ...
      }
    ]
  }
}
```

Processing rules:

1. Each document is verified according to its tier
2. `provenance_summary` reports per-document tier and status
3. If `allow_mixed_tiers: false`, all documents must match `default_tier`

---

## 5. Registry and Key Discovery

### 5.1 Supported Profiles

AIPE supports multiple key discovery mechanisms:

| Profile | URI Pattern | Use Case |
|---------|-------------|----------|
| `dpkr` | `/.well-known/dpkr.json` | Full lifecycle semantics |
| `jwks` | `/.well-known/jwks.json` | OIDC-compatible environments |
| `x509` | Certificate chain in document | Enterprise PKI |
| `did` | DID document resolution | Decentralized identity |

### 5.2 Offline Verification

For air-gapped environments, documents MAY embed sufficient key material:

```json
{
  "signature": {
    "algorithm": "ES256",
    "value": "<base64url>",
    "embedded_key": {
      "kty": "EC",
      "crv": "P-256",
      "x": "<base64url>",
      "y": "<base64url>"
    },
    "certificate_chain": ["<base64>"]
  }
}
```

When `embedded_key` is present:

- Verification MAY proceed without registry fetch
- Revocation checks are skipped (or best-effort if network available)
- Response includes `"verification_mode": "offline"`

---

## 6. Response Schema

### 6.1 Provenance Summary (All Tiers)

```json
{
  "provenance_summary": {
    "tier": "5.1",
    "verification_mode": "async",
    "documents_submitted": 5,
    "documents_verified": 4,
    "documents_failed": 1,
    "documents_excluded": 0,
    "failed_documents": [
      {
        "id": "doc_3",
        "reason": "signature_invalid",
        "issuer": "unknown.source.com"
      }
    ],
    "verified_issuers": ["research.acme.com", "legal.partner.org"],
    "verification_latency_ms": 45
  }
}
```

### 6.2 Citations with Provenance

Citations reference verified documents:

```json
{
  "citations": [
    {
      "start_index": 0,
      "end_index": 45,
      "document_id": "doc_1",
      "tier": "5.1",
      "verified": true,
      "issuer": "research.acme.com"
    }
  ]
}
```

### 6.3 Model Attestation (Tier 5.2+)

When `include_attestation: true`:

```json
{
  "model_attestation": {
    "provider": "api.anthropic.com",
    "model_id": "claude-3-opus-20240229",
    "attestation_type": "accountability_receipt",
    "signature": "<base64url>",
    "certificate_ref": "..."
  }
}
```

---

## 7. Streaming Extensions

### 7.1 Event Types by Tier

| Event | Tier 5.1 | Tier 5.2 | Tier 5.3 | Tier 5.4 |
|-------|----------|----------|----------|----------|
| `document_verification_complete` | ✓ | - | ✓ | ✓ |
| `document_verification_failed` | ✓ | - | ✓ | ✓ |
| `document_decrypted` | - | - | - | ✓ |
| `model_attestation` | - | ✓ | ✓ | ✓ |

### 7.2 Async Verification Events (Tier 5.1)

```
event: message_start
data: {"id": "msg_123"}

event: content_block_start
data: {"type": "text"}

event: content_block_delta
data: {"text": "Based on the report..."}

event: document_verification_complete
data: {"document_id": "doc_1", "status": "verified", "issuer": "research.acme.com"}

event: content_block_stop
data: {}

event: message_stop
data: {}
```

Note: In Tier 5.1 async mode, content streams before verification completes. Verification events arrive when ready.

---

## 8. Error Handling

### 8.1 Error Types

All tiers share common error types (RFC 7807):

| Error Type | HTTP Status | Tiers | Description |
|------------|-------------|-------|-------------|
| `signature-invalid` | 422 | 5.1, 5.3 | Signature verification failed |
| `certificate-expired` | 422 | All | Signing certificate has expired |
| `issuer-not-authorized` | 403 | All | Issuer not in allowlist |
| `binding-mismatch` | 422 | 5.1, 5.3 | Content hash doesn't match signature |
| `decryption-failed` | 422 | 5.4 | SPEVR envelope decryption failed |
| `key-not-found` | 422 | 5.4 | Issuer key not in registry |
| `verification-timeout` | 504 | 5.3, 5.4 | Verification exceeded time limit |

### 8.2 Tier-Specific Behavior

| Error | Tier 5.1 (Async) | Tier 5.3 (Sync) | Tier 5.4 (Enforced) |
|-------|------------------|-----------------|---------------------|
| Signature invalid | Document flagged, response continues | Request fails | Request fails |
| Certificate expired | Document flagged, response continues | Request fails | Request fails |
| Timeout | Document flagged, response continues | Configurable | Request fails |

---

## 9. Privacy Profiles

### 9.1 Profile Definitions

| Profile | Issuer Exposure | Model Exposure | Use Case |
|---------|-----------------|----------------|----------|
| `full` | Domain name | Full version | Audit environments |
| `hashed_issuer` | SHA-256 hash | Full version | Competitive sensitivity |
| `hashed_model` | Domain name | Family only | Operational security |
| `minimal` | SHA-256 hash | Family only | Maximum privacy |

### 9.2 Request Parameter

```json
{
  "provenance": {
    "privacy_profile": "hashed_issuer"
  }
}
```

### 9.3 Response with Hashed Issuer

```json
{
  "provenance_summary": {
    "verified_issuers": [
      "sha256:a1b2c3d4e5f6..."
    ],
    "issuer_count": 2
  }
}
```

---

## 10. C2PA Interoperability

### 10.1 C2PA Manifest as Signed Document

C2PA manifests are accepted as Tier 5.1 signed documents:

```json
{
  "type": "signed_document",
  "content": {
    "type": "image",
    "source": {"type": "base64", "data": "<base64>"}
  },
  "signature": {
    "type": "c2pa_manifest",
    "manifest_store": "<base64 JUMBF>"
  }
}
```

### 10.2 Verification Mapping

| C2PA Concept | AIPE Equivalent |
|--------------|-----------------|
| Claim Signature | `signature.value` |
| Hard Binding (hash) | `hard_binding.digest` |
| Soft Binding (watermark) | `soft_binding` |
| Trust List | `allowed_issuers` |

### 10.3 Limitations

C2PA manifests are **signature-only** (Tiers 5.1/5.3). They cannot be used for Tier 5.4 enforced provenance, which requires SPEVR envelopes.

---

## 11. Security Considerations

### 11.1 Tier Selection Guidance

| Threat | Recommended Tier |
|--------|------------------|
| Content tampering | 5.1+ (any signature-based) |
| Source attribution | 5.1+ |
| Compliance audit trail | 5.3 (sync verification) |
| Unauthorized viewing | 5.4 only |
| Model substitution | 5.2+ |

### 11.2 Tier 5.4 Specific Risks

- **Key loss**: Implement secure escrow and backup procedures
- **Performance**: Budget 300-500ms additional latency for k=500
- **Rotation overhead**: Use hierarchical key management

### 11.3 SSRF Mitigation

For documents with `ref_uri` (referenced payloads):

- Providers SHOULD reject `ref_uri` by default
- If supported, `ref_uri` MUST use `https://` only
- Private IP ranges MUST be blocked
- Maximum fetch size: 10MB
- Redirects SHOULD NOT be followed

### 11.4 Attestation Trust Model

Model attestation (Tier 5.2) provides **accountability**, not **execution proof**:

- It proves the provider signed a receipt claiming model X generated the response
- It does NOT prove model X actually executed
- Execution proof requires TEE attestation (out of scope)

---

## 12. Implementation Guidance

### 12.1 Recommended Adoption Path

1. **Start with Tier 5.1**: Implement async signature verification for RAG documents
2. **Add Tier 5.2**: Implement model attestation for accountability
3. **Graduate to Tier 5.3**: Enable sync verification for regulated workloads
4. **Deploy Tier 5.4 selectively**: Use only for genuinely confidential content

### 12.2 Performance Budgets

| Tier | Target Latency Overhead | Acceptable Range |
|------|------------------------|------------------|
| 5.1 (async) | 0ms (non-blocking) | 0-10ms |
| 5.2 | 5-10ms | 5-20ms |
| 5.3 | 50-100ms | 50-200ms |
| 5.4 | 200-400ms | 200-600ms |

### 12.3 Tier Capability Advertisement

Providers SHOULD advertise supported tiers:

```json
GET /.well-known/aipe-capabilities.json

{
  "supported_tiers": ["5.1", "5.2", "5.3"],
  "default_tier": "5.1",
  "supported_signature_algorithms": ["ES256", "ES384", "Ed25519"],
  "supported_encryption_suites": [],
  "max_documents_per_request": 100,
  "max_verification_time_ms": 1000
}
```

---

## 13. IANA Considerations

### 13.1 Media Type

No new media types. Tier 5.4 uses existing `application/spe+cbor`.

### 13.2 Error Type Registry

Register under `https://api.iaai.org/errors/`:

- `signature-invalid`
- `certificate-expired`
- `issuer-not-authorized`
- `binding-mismatch`
- `decryption-failed`
- `key-not-found`
- `verification-timeout`

### 13.3 Well-Known URI

Register `/.well-known/aipe-capabilities.json` for capability discovery.

---

## 14. Relationship to Other Specifications

| Specification | Relationship |
|---------------|--------------|
| **C2PA** | Tier 5.1/5.3 signature format interoperability |
| **SPET** | Tier 5.4 envelope format |
| **DPKR** | Key registry (one of several supported profiles) |
| **CDPN** | Quality Indicators (separate extension, not normative) |
| **Dream API** | Base API schema that AIPE extends |

---

## 15. Quality Indicators Extension

Quality Indicators are **moved to a separate specification**: `draft-iaai-aipe-qi-00`.

QI is:

- OPTIONAL for all AIPE tiers
- Non-normative for compliance
- Explicitly non-authoritative and non-comparable across providers

Implementations seeking QI functionality SHOULD reference the QI extension.

---

## Appendix A: Decision Tree for Tier Selection

```
Is the content confidential?
├── Yes: Is unauthorized viewing a security breach?
│   ├── Yes → Tier 5.4 (Enforced Provenance)
│   └── No → Tier 5.3 (Verified Context) with access control at application layer
└── No: Is the content subject to compliance requirements?
    ├── Yes: Must verification complete before processing?
    │   ├── Yes → Tier 5.3 (Verified Context)
    │   └── No → Tier 5.1 (Signed Context) with audit logging
    └── No: Is source attribution important?
        ├── Yes → Tier 5.1 (Signed Context)
        └── No → No AIPE tier required (plain documents)

Do you need model accountability?
├── Yes → Add Tier 5.2 (Attested Output) to above selection
└── No → Use above selection only
```

---

## Appendix B: Migration from draft-00

Implementations of draft-00 (encryption-only) map to draft-01 as follows:

| draft-00 Feature | draft-01 Equivalent |
|------------------|---------------------|
| `require_verification: true` | Tier 5.4 or Tier 5.3 |
| `spevr_envelope` | Tier 5.4 `spevr_envelope` (unchanged) |
| `model_attestation` (VRF) | Tier 5.2 with `attestation_algorithm: "vrf"` |
| `quality_indicators` | Separate QI extension |
| `reject_unverified: false` | Tier 5.1 async mode |

---

## Appendix C: Example Request (Tier 5.1 + 5.2)

A typical production configuration: signed documents with async verification and model attestation.

```json
{
  "model": "claude-3-opus-20240229",
  "provenance": {
    "tier": "5.1",
    "verification_mode": "async",
    "include_attestation": true,
    "allowed_issuers": ["research.acme.com", "*.partner.org"],
    "privacy_profile": "full"
  },
  "context": {
    "system": "You are a financial analyst. Cite sources.",
    "documents": [
      {
        "id": "q4_report",
        "type": "signed_document",
        "content": {
          "type": "text",
          "text": "Q4 2024 Financial Summary..."
        },
        "signature": {
          "algorithm": "ES256",
          "value": "MEUCIQDx...",
          "certificate_chain": ["MIIB..."]
        },
        "hard_binding": {
          "algorithm": "SHA-256",
          "digest": "abc123..."
        }
      }
    ],
    "citations_required": true
  },
  "messages": [
    {
      "role": "user",
      "content": "Summarize the Q4 revenue trends."
    }
  ]
}
```

---

## Appendix D: Example Response (Tier 5.1 + 5.2)

```json
{
  "id": "msg_resp_456",
  "created": 1703347200,
  "model": "claude-3-opus-20240229",
  "provenance_summary": {
    "tier": "5.1",
    "verification_mode": "async",
    "documents_submitted": 1,
    "documents_verified": 1,
    "verification_results": [
      {
        "document_id": "q4_report",
        "status": "verified",
        "issuer": "research.acme.com",
        "binding_intact": true
      }
    ],
    "verification_latency_ms": 23
  },
  "model_attestation": {
    "provider": "api.anthropic.com",
    "model_id": "claude-3-opus-20240229",
    "attestation_type": "accountability_receipt",
    "algorithm": "ES256",
    "payload": {
      "request_hash": "sha256:def456...",
      "response_hash": "sha256:ghi789...",
      "timestamp": 1703347200
    },
    "signature": "MEQCIG..."
  },
  "usage": {
    "input_tokens": 1247,
    "output_tokens": 342
  },
  "choices": [
    {
      "index": 0,
      "finish_reason": "stop",
      "citations": [
        {
          "start_index": 0,
          "end_index": 52,
          "document_id": "q4_report",
          "verified": true,
          "issuer": "research.acme.com"
        }
      ],
      "message": {
        "role": "assistant",
        "content": "According to the Q4 2024 Financial Summary, revenue increased 15% year-over-year, driven primarily by..."
      }
    }
  ]
}
```

---

**End of draft-iaai-aipe-01**
