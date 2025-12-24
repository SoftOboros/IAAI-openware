# Internet-Draft: AI API Provenance Extension (AIPE)

**Intended status:** Standards Track  
**Expires:** 2026-06-23  
**Version:** draft-iaai-aipe-00

---

## Abstract

This document defines the **AI API Provenance Extension (AIPE)**: a specification layer that integrates cryptographic provenance verification into standardized AI API interfaces. AIPE bridges the Domain Provenance Key Registry (DPKR), Secure Provenance Envelope Transport (SPET), and Coherence-Driven Provenance Network (CDPN) specifications with the emerging "Dream API" unified AI interface standard.

AIPE addresses three critical gaps in current AI API designs: (1) the absence of cryptographic verification for RAG context documents, enabling data poisoning attacks; (2) the lack of model identity attestation, preventing auditability of which model generated a response; and (3) the absence of standardized quality feedback mechanisms for semantic coherence evaluation.

This specification defines new content block types, response metadata structures, error codes, and streaming events that enable **enforced provenance** at the API boundary—ensuring that AI systems cannot consume unverified data and that AI outputs carry cryptographic proof of their origin.

---

## Status of This Memo

This Internet-Draft is submitted in full conformance with the provisions of BCP 78 and BCP 79. Distribution of this memo is unlimited.

---

## 1. Introduction

### 1.1 The Trust Gap in AI APIs

The current generation of AI APIs treats provenance as an afterthought. Documents injected into RAG pipelines carry no cryptographic binding to their source. Model responses provide no verifiable attestation of which model version generated them. Quality metrics, when present, measure only token consumption rather than semantic integrity.

This architecture creates three categories of vulnerability:

1. **Data Poisoning**: Attackers inject malicious documents into vector databases. Without provenance verification, AI systems cannot distinguish legitimate sources from adversarial injections.

2. **Model Substitution**: When requests route through aggregators or proxies, there is no cryptographic proof that the claimed model actually processed the request. A provider could substitute a cheaper model while billing for a premium one.

3. **Coherence Blindness**: Current APIs provide no standardized mechanism for downstream systems to signal whether retrieved content was actually useful, preventing the self-tuning feedback loops that enable healthy information ecosystems.

### 1.2 The Solution: Enforced Provenance at the API Boundary

AIPE integrates the provenance stack (DPKR/SPET/CDPN) into AI API contracts such that:

- **Input verification is mandatory**: Documents wrapped in SPET envelopes MUST be cryptographically verified before injection into model context.
- **Output attestation is standard**: Responses include SPET binding objects proving model identity.
- **Quality feedback is structured**: CDPN Quality Indicators become first-class response metadata.

The result is an AI API where the act of using data is mathematically inseparable from verifying its origin—the principle of **enforced provenance**.

### 1.3 Relationship to Other Specifications

| Specification | Role in AIPE |
|---------------|--------------|
| **DPKR** (draft-iaai-dpkr-00) | Defines key registries for both document issuers and model providers |
| **SPET** (draft-iaai-spet-transport-00) | Defines envelope format for verified content blocks and response attestations |
| **CDPN** (draft-iaai-cdpn-00) | Defines Quality Indicator feedback semantics |
| **Dream API** (draft-iaai-dreamapi-00) | Defines base AI API schema that AIPE extends |

AIPE is a **profile specification** that binds these components into a coherent AI API integration.

---

## 2. Terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in RFC 2119 and RFC 8174.

* **Provenance-Aware Endpoint**: An AI API endpoint implementing AIPE Level 5 compliance.
* **Verified Content Block**: A content block whose SPET envelope has passed cryptographic verification.
* **Model Attestation**: A SPET binding object proving which model generated a response.
* **Quality Indicator (QI)**: A scalar feedback value per CDPN specification.
* **Blind Retrieval**: RAG architecture where the API layer cannot read document content without successful provenance verification.

---

## 3. Compliance Level: Level 5 (Provenance-Aware)

AIPE defines **Level 5** compliance for the Dream API specification, building upon Levels 1-4.

### 3.1 Prerequisites

A Level 5 compliant endpoint MUST first satisfy:

- **Level 1** (Foundation): Unified messages endpoint, RFC 7807 errors
- **Level 2** (Structured & Tooling): JSON Schema tools, structured output
- **Level 3** (Multimodal & Contextual): Unified content blocks, documents parameter
- **Level 4** (Realtime & Stateful): WebSocket/SSE streaming, session management

### 3.2 Level 5 Requirements

A Level 5 compliant endpoint MUST implement:

1. **Input Verification**: Support for SPEVR-wrapped documents with mandatory verification
2. **Output Attestation**: SPET binding objects in all responses
3. **Registry Integration**: DPKR-compliant key registry publication
4. **Quality Feedback**: CDPN Quality Indicator response metadata
5. **Provenance Errors**: RFC 7807 error types for provenance failures
6. **Streaming Events**: Provenance-specific SSE event types

---

## 4. Request Schema Extensions

### 4.1 The `provenance` Configuration Object

AIPE adds a top-level `provenance` object to the Dream API request schema:

```json
{
  "model": "vendor-model-id",
  "provenance": {
    "require_verification": true,
    "allowed_issuers": ["research.acme.com", "legal.partner.org"],
    "allowed_suites": ["SPE-VRF-EDWARDS25519-SHA512-TAI-AES256GCM"],
    "reject_unverified": true,
    "include_attestation": true,
    "quality_feedback_enabled": true
  },
  "context": { ... },
  "messages": [ ... ]
}
```

#### 4.1.1 `require_verification` (boolean, REQUIRED for Level 5)

When `true`, all documents in the `context.documents` array that are wrapped in SPET envelopes MUST be cryptographically verified before context injection.

Default: `false` (for backward compatibility with Levels 1-4)

#### 4.1.2 `allowed_issuers` (array of strings, OPTIONAL)

Whitelist of `issuer_domain` values. Documents from unlisted domains MUST be rejected.

If omitted, all domains with valid DPKR registries are permitted.

#### 4.1.3 `allowed_suites` (array of strings, OPTIONAL)

Whitelist of SPET `suite_id` values. Envelopes using unlisted suites MUST be rejected.

If omitted, all suites supported by the endpoint are permitted.

#### 4.1.4 `reject_unverified` (boolean, OPTIONAL)

When `true`, the entire request MUST fail if any SPET envelope fails verification.

When `false`, unverified documents are silently excluded from context.

Default: `true`

#### 4.1.5 `include_attestation` (boolean, OPTIONAL)

When `true`, the response MUST include a `model_attestation` object.

Default: `true` for Level 5 endpoints

#### 4.1.6 `quality_feedback_enabled` (boolean, OPTIONAL)

When `true`, the response MUST include `quality_indicators` metadata.

Default: `false`

---

### 4.2 Extended Content Block: `spevr_envelope`

AIPE defines a new content block type for SPET-wrapped content:

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
  },
  "registry_hint": "https://research.acme.com/.well-known/dpkr.json"
}
```

#### 4.2.1 Processing Requirements

Upon receiving an `spevr_envelope` content block:

1. Resolve DPKR registry for `issuer_domain`
2. Validate `issuer_kid` against registry
3. Enforce `suite_id` against `allowed_suites`
4. Derive `K_sym` per SPET suite specification
5. AEAD decrypt payload
6. Inject plaintext into model context

Failure at any step MUST trigger provenance error handling per Section 7.

#### 4.2.2 Inline vs. Referenced Envelopes

For large documents, the envelope MAY use SPET's referenced payload mode:

```json
{
  "type": "spevr_envelope",
  "envelope": {
    "payload": {
      "ref_uri": "https://storage.acme.com/docs/report.enc",
      "ref_hash": "<base64url>",
      "iv": "<base64url>",
      "tag": "<base64url>",
      "ptype": "application/pdf"
    }
  }
}
```

The endpoint MUST fetch and verify the referenced ciphertext.

---

### 4.3 Extended `context.documents` Array

The Dream API's `documents` parameter is extended to support mixed verified and unverified content:

```json
{
  "context": {
    "documents": [
      {
        "id": "doc_1",
        "type": "text",
        "content": "Unverified plain text document..."
      },
      {
        "id": "doc_2",
        "type": "spevr_envelope",
        "envelope": { ... }
      },
      {
        "id": "doc_3",
        "type": "spevr_envelope",
        "envelope": { ... }
      }
    ],
    "require_verification": true,
    "citations_required": true
  }
}
```

When `require_verification` is `true`:

- Documents of type `text`, `url`, or `file` without SPET wrapping MUST be rejected
- Only `spevr_envelope` documents are permitted

---

## 5. Response Schema Extensions

### 5.1 The `model_attestation` Object

Level 5 responses MUST include a `model_attestation` object proving model identity:

```json
{
  "id": "msg_xyz789",
  "created": 1703347200,
  "model_attestation": {
    "provider_domain": "api.openai.com",
    "model_id": "gpt-4-turbo-2024-01-25",
    "model_kid": "gpt4t-signing-key-2024q1",
    "suite_id": "SPE-VRF-P256-SHA256-TAI-AES256GCM",
    "binding": {
      "mode": "vrf_vdkd",
      "input": "<base64url-hash-of-request>",
      "proof": "<base64url>"
    },
    "attested_at": 1703347201,
    "registry_uri": "https://api.openai.com/.well-known/dpkr.json"
  },
  "choices": [ ... ]
}
```

#### 5.1.1 Attestation Semantics

The `model_attestation.binding.input` MUST be computed as:

```
input = SHA-256(
  request_id ||
  model_id ||
  SHA-256(canonical_request_body) ||
  created_timestamp
)
```

This binds the attestation to the specific request, preventing replay.

#### 5.1.2 Verification by Clients

Clients MAY verify the attestation by:

1. Fetching the provider's DPKR registry
2. Resolving `model_kid`
3. Verifying the VRF proof
4. Confirming the derived `input` matches expected values

This provides cryptographic proof that the claimed model processed the request.

---

### 5.2 The `provenance_summary` Object

Responses MUST include a summary of provenance verification results:

```json
{
  "provenance_summary": {
    "documents_submitted": 5,
    "documents_verified": 4,
    "documents_rejected": 1,
    "verified_issuers": [
      "research.acme.com",
      "legal.partner.org"
    ],
    "rejected_documents": [
      {
        "id": "doc_3",
        "reason": "key_expired",
        "issuer_domain": "old.vendor.com"
      }
    ]
  }
}
```

---

### 5.3 The `quality_indicators` Object

When `quality_feedback_enabled` is `true`, responses include CDPN-style quality metrics:

```json
{
  "quality_indicators": {
    "overall_coherence": 0.87,
    "dimensions": {
      "relevance": 0.92,
      "grounding_coverage": 0.85,
      "citation_accuracy": 0.91,
      "non_contradiction": 0.88,
      "novelty": 0.72
    },
    "per_document": [
      {
        "document_id": "doc_1",
        "coherence": 0.94,
        "citations_used": 3
      },
      {
        "document_id": "doc_2",
        "coherence": 0.78,
        "citations_used": 1
      }
    ],
    "evaluated_against": "request_context"
  }
}
```

#### 5.3.1 Quality Indicator Semantics

Per CDPN specification:

- `1.0` indicates maximal coherence
- `0.0` indicates incoherence or rejection
- Threshold interpretation is receiver-local

#### 5.3.2 Feedback Loop Integration

Clients implementing CDPN sender-side adaptation SHOULD:

- Associate QI values with `(provider_domain, model_id)` pairs
- Adjust model selection based on sustained QI patterns
- NOT treat QI as a global reputation score

---

### 5.4 Extended `citations` Array

Citations MUST reference verified envelope IDs when citing SPEVR documents:

```json
{
  "choices": [
    {
      "message": {
        "content": [
          {
            "type": "text",
            "text": "According to the Q4 analysis, revenue increased by 15%."
          }
        ]
      },
      "citations": [
        {
          "start_index": 15,
          "end_index": 52,
          "document_id": "doc_2",
          "envelope_id": "spe_abc123",
          "issuer_domain": "research.acme.com",
          "verified": true
        }
      ]
    }
  ]
}
```

The `envelope_id` and `verified` fields enable downstream systems to audit citation provenance.

---

## 6. Streaming Extensions

### 6.1 New SSE Event Types

AIPE defines additional Server-Sent Event types for provenance-aware streaming:

#### 6.1.1 `provenance_verification_start`

Emitted when envelope verification begins:

```
event: provenance_verification_start
data: {"envelope_id": "spe_abc123", "issuer_domain": "research.acme.com"}
```

#### 6.1.2 `provenance_verified`

Emitted when an envelope passes verification:

```
event: provenance_verified
data: {
  "envelope_id": "spe_abc123",
  "issuer_domain": "research.acme.com",
  "issuer_kid": "key-2024-q4",
  "verified": true,
  "verified_at": 1703347200
}
```

#### 6.1.3 `provenance_rejected`

Emitted when an envelope fails verification:

```
event: provenance_rejected
data: {
  "envelope_id": "spe_xyz789",
  "issuer_domain": "untrusted.source.com",
  "reason": "issuer_not_in_allowlist",
  "fatal": true
}
```

If `fatal` is `true` and `reject_unverified` is `true`, the stream terminates.

#### 6.1.4 `quality_indicator`

Emitted at stream end with coherence metrics:

```
event: quality_indicator
data: {
  "overall_coherence": 0.87,
  "grounding_coverage": 0.85,
  "documents_cited": ["doc_1", "doc_2"]
}
```

#### 6.1.5 `model_attestation`

Emitted at stream end with model identity proof:

```
event: model_attestation
data: {
  "provider_domain": "api.anthropic.com",
  "model_id": "claude-3-opus-20240229",
  "binding": { ... }
}
```

### 6.2 Event Ordering

Provenance events MUST be emitted in the following order:

1. `provenance_verification_start` (one per envelope)
2. `provenance_verified` or `provenance_rejected` (one per envelope)
3. `content_block_start` (per Dream API Level 4)
4. `content_block_delta` (streaming content)
5. `content_block_stop`
6. `quality_indicator` (if enabled)
7. `model_attestation` (if enabled)
8. `message_stop`

---

## 7. Error Handling

### 7.1 Provenance Error Types

AIPE defines RFC 7807 error types for provenance failures:

#### 7.1.1 `provenance-verification-failed`

```json
{
  "type": "https://api.iaai.org/errors/provenance-verification-failed",
  "title": "Document Provenance Verification Failed",
  "status": 422,
  "detail": "SPEVR envelope 'spe_xyz789' failed VRF verification against issuer 'research.acme.com'",
  "instance": "/v1/interactions/req_abc123",
  "extensions": {
    "envelope_id": "spe_xyz789",
    "issuer_domain": "research.acme.com",
    "issuer_kid": "key-2024-q4",
    "failure_step": "vrf_verify",
    "suite_id": "SPE-VRF-EDWARDS25519-SHA512-TAI-AES256GCM"
  }
}
```

#### 7.1.2 `issuer-key-not-found`

```json
{
  "type": "https://api.iaai.org/errors/issuer-key-not-found",
  "title": "Issuer Key Not Found in Registry",
  "status": 422,
  "detail": "Key 'key-2024-q4' not found in DPKR registry for domain 'research.acme.com'",
  "extensions": {
    "issuer_domain": "research.acme.com",
    "issuer_kid": "key-2024-q4",
    "registry_uri": "https://research.acme.com/.well-known/dpkr.json",
    "registry_fetched_at": 1703347200
  }
}
```

#### 7.1.3 `issuer-domain-not-authorized`

```json
{
  "type": "https://api.iaai.org/errors/issuer-domain-not-authorized",
  "title": "Issuer Domain Not in Allowlist",
  "status": 403,
  "detail": "Domain 'untrusted.vendor.com' is not in the allowed_issuers list",
  "extensions": {
    "issuer_domain": "untrusted.vendor.com",
    "allowed_issuers": ["research.acme.com", "legal.partner.org"]
  }
}
```

#### 7.1.4 `envelope-expired`

```json
{
  "type": "https://api.iaai.org/errors/envelope-expired",
  "title": "Provenance Envelope Expired",
  "status": 422,
  "detail": "Envelope 'spe_abc123' expired at 2024-01-01T00:00:00Z",
  "extensions": {
    "envelope_id": "spe_abc123",
    "expires_at": 1704067200,
    "current_time": 1704153600
  }
}
```

#### 7.1.5 `suite-not-supported`

```json
{
  "type": "https://api.iaai.org/errors/suite-not-supported",
  "title": "Cryptographic Suite Not Supported",
  "status": 422,
  "detail": "Suite 'SPE-PQC-MLKEM768-MLDSA65-AES256GCM' is not supported by this endpoint",
  "extensions": {
    "requested_suite": "SPE-PQC-MLKEM768-MLDSA65-AES256GCM",
    "supported_suites": [
      "SPE-VRF-EDWARDS25519-SHA512-TAI-AES256GCM",
      "SPE-VRF-P256-SHA256-TAI-AES256GCM"
    ]
  }
}
```

#### 7.1.6 `key-compromised`

```json
{
  "type": "https://api.iaai.org/errors/key-compromised",
  "title": "Issuer Key Marked as Compromised",
  "status": 422,
  "detail": "Key 'key-2023-q2' was compromised at 2023-09-15T00:00:00Z; envelope created after compromise",
  "extensions": {
    "issuer_kid": "key-2023-q2",
    "compromised_at": 1694736000,
    "envelope_created_at": 1695340800
  }
}
```

#### 7.1.7 `registry-unreachable`

```json
{
  "type": "https://api.iaai.org/errors/registry-unreachable",
  "title": "DPKR Registry Unreachable",
  "status": 502,
  "detail": "Unable to fetch DPKR registry for domain 'research.acme.com'",
  "extensions": {
    "issuer_domain": "research.acme.com",
    "registry_uri": "https://research.acme.com/.well-known/dpkr.json",
    "last_attempt": 1703347200,
    "cached_registry_expired": true
  }
}
```

---

## 8. Provider Requirements

### 8.1 DPKR Registry Publication

Level 5 compliant providers MUST publish a DPKR registry containing keys for each model version:

```json
{
  "domain": "api.anthropic.com",
  "registry_version": "1.0",
  "published_at": 1703347200,
  "keys": [
    {
      "kid": "claude-3-opus-signing-2024q1",
      "public_key": "<JWK>",
      "key_type": "vrf",
      "allowed_suites": ["SPE-VRF-P256-SHA256-TAI-AES256GCM"],
      "status": "active",
      "valid_from": 1704067200,
      "metadata": {
        "model_id": "claude-3-opus-20240229",
        "model_family": "claude-3"
      }
    }
  ],
  "policy": {
    "default_suites": ["SPE-VRF-P256-SHA256-TAI-AES256GCM"],
    "cache_ttl": 3600
  }
}
```

### 8.2 Model-to-Key Binding

Each model version MUST have a dedicated signing key. Key reuse across model versions is PROHIBITED.

### 8.3 Attestation Generation

Providers MUST generate attestations using the model's dedicated VRF key at response generation time, not at a proxy or gateway layer.

---

## 9. Security Considerations

### 9.1 Blind Retrieval Enforcement

When `require_verification` is `true`, endpoints MUST NOT inject plaintext into model context until SPET verification succeeds. This prevents timing attacks where partial content exposure occurs before verification completes.

### 9.2 Registry Caching

Endpoints MAY cache DPKR registries per `cache_ttl`. However:

- Cached registries MUST NOT be used beyond their TTL
- Key revocations MUST be honored immediately upon registry refresh
- If a registry is unreachable and cache is expired, requests MUST fail closed

### 9.3 Model Attestation Replay

Attestations are bound to specific requests via the `input` hash. Clients MUST verify that the `input` matches their request before trusting the attestation.

### 9.4 Quality Indicator Gaming

QI values are informational and receiver-computed. Providers could theoretically inflate QI scores. Clients SHOULD:

- Perform independent coherence evaluation when critical
- Use QI as one signal among many
- Monitor for QI drift over time

---

## 10. Privacy Considerations

### 10.1 Issuer Domain Exposure

The `provenance_summary` reveals which document issuers were used. This may leak information about data sources. Providers MAY offer a `redact_issuer_domains` option.

### 10.2 Model Selection Inference

Detailed `model_attestation` reveals exact model versions. Some deployments may prefer coarse-grained attestation (e.g., "claude-3-opus" without version suffix).

---

## 11. IANA Considerations

This document requests:

### 11.1 Error Type Registry

Registration of error type URIs under `https://api.iaai.org/errors/`:

- `provenance-verification-failed`
- `issuer-key-not-found`
- `issuer-domain-not-authorized`
- `envelope-expired`
- `suite-not-supported`
- `key-compromised`
- `registry-unreachable`

### 11.2 SSE Event Type Registry

Registration of SSE event types:

- `provenance_verification_start`
- `provenance_verified`
- `provenance_rejected`
- `quality_indicator`
- `model_attestation`

### 11.3 Content Block Type Registry

Registration of content block type:

- `spevr_envelope`

---

## 12. Implementation Guidance

### 12.1 Migration Path

Providers transitioning from Level 4 to Level 5:

1. **Phase 1**: Publish DPKR registry with model signing keys
2. **Phase 2**: Support `spevr_envelope` content blocks (optional verification)
3. **Phase 3**: Support `model_attestation` in responses
4. **Phase 4**: Enable `require_verification` mode
5. **Phase 5**: Full Level 5 compliance certification

### 12.2 Client Integration

Clients adopting AIPE:

1. Wrap RAG documents in SPEVR envelopes at ingestion time
2. Set `require_verification: true` in requests
3. Verify `model_attestation` in responses
4. Implement CDPN sender-side adaptation using QI feedback

### 12.3 Reference Implementation

The IAAI SHALL provide:

- **Dream Shim + AIPE**: Reference proxy implementing Level 5 atop existing providers
- **SPEVR SDK**: Libraries for envelope creation/verification in Python, TypeScript, Go
- **DPKR Publisher**: Tools for generating and hosting compliant registries

---

## 13. Relationship to Existing Standards

### 13.1 C2PA Integration

AIPE complements C2PA by providing cryptographic enforcement where C2PA provides metadata attachment. A document MAY carry both:

- C2PA manifest (human-readable provenance history)
- SPEVR envelope (cryptographically enforced access control)

### 13.2 OpenID Connect

The Dream API's OIDC authentication is orthogonal to AIPE. OIDC authenticates the *client*; AIPE authenticates *content* and *model responses*.

### 13.3 JSON Web Tokens

SPEVR envelopes are not JWTs but serve a similar attestation purpose. Future work MAY define JWT-based envelope formats for environments preferring JOSE tooling.

---

## Appendix A: Complete Request Example

```json
{
  "model": "claude-3-opus-20240229",
  "provenance": {
    "require_verification": true,
    "allowed_issuers": ["research.acme.com", "legal.partner.org"],
    "allowed_suites": ["SPE-VRF-EDWARDS25519-SHA512-TAI-AES256GCM"],
    "reject_unverified": true,
    "include_attestation": true,
    "quality_feedback_enabled": true
  },
  "config": {
    "temperature": 0.7,
    "max_tokens": 4096,
    "stream": true
  },
  "context": {
    "system": "You are a financial analyst. Only cite verified documents.",
    "documents": [
      {
        "id": "doc_q4_report",
        "type": "spevr_envelope",
        "envelope": {
          "spe_version": "1.0",
          "id": "spe_financial_q4_2024",
          "issuer_domain": "research.acme.com",
          "issuer_kid": "analyst-key-2024",
          "suite_id": "SPE-VRF-EDWARDS25519-SHA512-TAI-AES256GCM",
          "created_at": 1703260800,
          "binding": {
            "mode": "vrf_vdkd",
            "input": "base64url...",
            "proof": "base64url..."
          },
          "payload": {
            "ciphertext": "base64url...",
            "iv": "base64url...",
            "tag": "base64url...",
            "ptype": "text/markdown"
          }
        }
      }
    ],
    "citations_required": true
  },
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Summarize the Q4 revenue trends from the verified report."
        }
      ]
    }
  ]
}
```

---

## Appendix B: Complete Response Example

```json
{
  "id": "msg_resp_abc123",
  "created": 1703347200,
  "model": "claude-3-opus-20240229",
  "model_attestation": {
    "provider_domain": "api.anthropic.com",
    "model_id": "claude-3-opus-20240229",
    "model_kid": "claude-3-opus-signing-2024q1",
    "suite_id": "SPE-VRF-P256-SHA256-TAI-AES256GCM",
    "binding": {
      "mode": "vrf_vdkd",
      "input": "base64url...",
      "proof": "base64url..."
    },
    "attested_at": 1703347201,
    "registry_uri": "https://api.anthropic.com/.well-known/dpkr.json"
  },
  "provenance_summary": {
    "documents_submitted": 1,
    "documents_verified": 1,
    "documents_rejected": 0,
    "verified_issuers": ["research.acme.com"]
  },
  "quality_indicators": {
    "overall_coherence": 0.92,
    "dimensions": {
      "relevance": 0.95,
      "grounding_coverage": 0.89,
      "citation_accuracy": 0.94,
      "non_contradiction": 0.91
    },
    "per_document": [
      {
        "document_id": "doc_q4_report",
        "coherence": 0.92,
        "citations_used": 4
      }
    ]
  },
  "usage": {
    "input_tokens": 2847,
    "output_tokens": 512,
    "cache_read_tokens": 0
  },
  "choices": [
    {
      "index": 0,
      "finish_reason": "stop",
      "citations": [
        {
          "start_index": 0,
          "end_index": 45,
          "document_id": "doc_q4_report",
          "envelope_id": "spe_financial_q4_2024",
          "issuer_domain": "research.acme.com",
          "verified": true
        }
      ],
      "message": {
        "role": "assistant",
        "content": [
          {
            "type": "text",
            "text": "Based on the verified Q4 financial report, revenue increased 15% year-over-year..."
          }
        ]
      }
    }
  ]
}
```

---

## Appendix C: Streaming Event Sequence Example

```
event: provenance_verification_start
data: {"envelope_id": "spe_financial_q4_2024", "issuer_domain": "research.acme.com"}

event: provenance_verified
data: {"envelope_id": "spe_financial_q4_2024", "issuer_domain": "research.acme.com", "verified": true}

event: message_start
data: {"id": "msg_resp_abc123", "model": "claude-3-opus-20240229"}

event: content_block_start
data: {"index": 0, "type": "text"}

event: content_block_delta
data: {"index": 0, "delta": {"type": "text_delta", "text": "Based on the verified Q4 "}}

event: content_block_delta
data: {"index": 0, "delta": {"type": "text_delta", "text": "financial report, revenue "}}

event: content_block_delta
data: {"index": 0, "delta": {"type": "text_delta", "text": "increased 15% year-over-year..."}}

event: content_block_stop
data: {"index": 0}

event: quality_indicator
data: {"overall_coherence": 0.92, "grounding_coverage": 0.89}

event: model_attestation
data: {"provider_domain": "api.anthropic.com", "model_id": "claude-3-opus-20240229", "binding": {...}}

event: message_stop
data: {"stop_reason": "end_turn"}
```

---

**End of draft-iaai-aipe-00**
