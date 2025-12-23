# Internet-Draft: Secure Provenance Envelope Transport (SPET)

**Intended status:** Standards Track  
**Expires:** 2026-06-23  
**Version:** draft-iaai-spet-transport-00

---

## Abstract

This document defines the **Secure Provenance Envelope (SPE)**: a **transport-agnostic**, cryptographically verifiable container for data transfer between **registered domains**. SPE allows a receiver to (1) authenticate the issuer’s provenance key via a domain public-key registry (defined elsewhere), and (2) decrypt an encrypted payload using a symmetric key that is **bound to issuer authenticity** and payload integrity.

SPE supports multiple cryptographic suite families, including **VRF-based verifiable deterministic key derivation (VDKD)** for *cryptographically enforced provenance*, and **post-quantum lattice modes** for *policy-enforced provenance* using NIST-standardized primitives (ML-KEM, ML-DSA, SLH-DSA).

---

## Status of This Memo

This Internet-Draft is submitted in full conformance with the provisions of BCP 78 and BCP 79. Distribution of this memo is unlimited.

---

## 1. Terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in RFC 2119 and RFC 8174.

* **Issuer Domain**: DNS domain asserting provenance authority for an envelope.
* **Receiver Domain**: Domain processing the envelope.
* **Domain Registry**: A domain-controlled registry that publishes authorized provenance keys and policy for that domain (dependency; out of scope).
* **issuer_kid**: Key identifier scoped to `issuer_domain`.
* **Binding Mode**: Mechanism used to derive or obtain the symmetric content key `K_sym`.

**Normative principle:** A receiver MUST treat the domain registry as the sole authority for issuer keys. Envelope-carried keys are non-authoritative unless they match a registry-authorized key.

---

## 2. Design Goals and Non-Goals

### 2.1 Goals

1. **Economical reject path:** Domain membership and key authorization checks occur before expensive cryptographic or AI processing (the “membrane” principle).
2. **Transport agnosticism:** SPE is a container and may be carried over any byte-preserving transport.
3. **Algorithm agility:** Cryptographic suites are extensible without changing the envelope framing.
4. **Enforced provenance option:** Certain suites ensure that decryption is mathematically contingent on successful provenance verification.

### 2.2 Non-Goals

1. Semantic truth evaluation or coherence scoring.
2. Definition of the domain registry mechanism.
3. Global public identity infrastructure.

---

## 3. Encoding and Media Types

### 3.1 Normative Encoding

SPE **MUST** be encoded using **Deterministic CBOR** (RFC 8949 deterministic encoding profile).

* Media type: `application/spe+cbor`
* All cryptographic operations (AEAD AAD binding, signature inputs, etc.) are defined over this deterministic CBOR encoding.

### 3.2 Optional JSON Wrapper (Transport Convenience Only)

A JSON representation **MAY** be used solely as a wrapper for canonical CBOR bytes.

* Media type: `application/spe+json`

```json
{
  "spe_cbor": "<base64url-encoded application/spe+cbor bytes>"
}
```

Receivers **MUST** ignore any JSON fields not represented in the canonical CBOR structure.

---

## 4. Envelope Data Model (CBOR Map)

Top-level CBOR map keys:

| Key | Type | Required | Description |
|---|---|---|---|
| `spe_version` | tstr | Yes | "1.0" |
| `id` | tstr | Yes | Unique envelope identifier (UUID v4 RECOMMENDED) |
| `issuer_domain` | tstr | Yes | Canonical DNS / IDNA domain |
| `issuer_kid` | tstr | Yes | Key identifier (registry-scoped) |
| `suite_id` | tstr | Yes | Cryptographic suite identifier |
| `created_at` | uint | Yes | Unix epoch seconds |
| `expires_at` | uint | No | Unix epoch seconds |
| `binding` | map | Yes | Binding object (Section 5) |
| `payload` | map | Yes | Payload object (Section 6) |
| `assertions` | map | No | Optional provenance metadata |

---

## 5. Binding Object

The `binding` CBOR map contains cryptographic material required to derive `K_sym`.

| Key | Type | Required | Description |
|---|---|---|---|
| `mode` | tstr | Yes | `vrf_vdkd`, `vrf_vdkd_kem`, `kem_sig_pqc`, `hybrid` |
| `input` | bstr | Yes | Suite-defined input (e.g., `SHA-256(plaintext)` or document identifier) |
| `proof` | bstr | Yes | Suite-defined proof blob |
| `aad_commit` | bstr | No | Optional commitment to canonical AAD bytes |

---

## 6. Payload Object

### 6.1 Inline Payload

| Key | Type | Required | Description |
|---|---|---|---|
| `ciphertext` | bstr | Yes | Encrypted payload |
| `iv` | bstr | Yes | AEAD nonce |
| `tag` | bstr | Yes | AEAD authentication tag |
| `ptype` | tstr | Yes | Informational payload type |

### 6.2 Referenced Payload (Large Object Option)

| Key | Type | Required | Description |
|---|---|---|---|
| `ref_uri` | tstr | Yes | URI to ciphertext object |
| `ref_hash` | bstr | Yes | Hash of referenced ciphertext |
| `iv` | bstr | Yes | AEAD nonce |
| `tag` | bstr | Yes | AEAD authentication tag |
| `ptype` | tstr | Yes | Informational payload type |

---

## 7. Canonical AEAD AAD

AEAD AAD **MUST** be the deterministic-CBOR encoding of the following map:

```
{
  spe_version,
  id,
  issuer_domain,
  issuer_kid,
  suite_id,
  created_at,
  expires_at (if present),
  binding.mode,
  binding.input
}
```

No additional fields may be included in AAD.

---

## 8. Cryptographic Suites

### 8.1 VRF / VDKD Suites (Cryptographically Enforced Provenance)

These suites implement the pattern:

1. Verify VRF proof using issuer public key
2. Derive `beta`
3. Derive `K_sym = SHA-256(beta)`
4. AEAD decrypt

Failure at any step **MUST** result in rejection.

Initial suite identifiers:

* `SPE-VRF-EDWARDS25519-SHA512-TAI-AES256GCM`
* `SPE-VRF-P256-SHA256-TAI-AES256GCM`
* `SPE-VRF-RSA2048-FDH-SHA256-PSS-AES256GCM` (legacy)

### 8.2 VRF + Receiver KEM (Confidentiality + Enforced Provenance)

* `mode = vrf_vdkd_kem`
* `K_sym = HKDF-SHA256(beta || ss, info="SPE-VRFKEM-v1", L=32)`

### 8.3 Post-Quantum Lattice Suites (Policy-Enforced Provenance)

* `mode = kem_sig_pqc`
* KEM: ML-KEM (FIPS 203)
* Signature: ML-DSA (FIPS 204) or SLH-DSA (FIPS 205)
* `K_sym = HKDF-SHA256(ss, info="SPE-PQC-v1", L=32)`

Signature verification is **REQUIRED** by protocol policy but does not mathematically gate key derivation.

### 8.4 Hybrid Suites (Migration)

* `mode = hybrid`
* Combines VRF enforced provenance with PQC signature attestation

---

## 9. Receiver Processing Algorithm (Normative)

1. Parse deterministic CBOR; reject unsupported version
2. Enforce expiry if present
3. Resolve `(issuer_domain, issuer_kid)` via domain registry
4. Enforce suite policy
5. Derive `K_sym` per suite
6. AEAD decrypt using canonical AAD
7. Perform post-decrypt binding checks
8. Accept plaintext into local pipeline

---

## 10. Security Considerations

* Registry authenticity is critical; MITM protection is REQUIRED at the registry layer
* Replay protection via `id`, `created_at`, and local deduplication
* DoS resistance relies on early membership and key checks
* Key compromise handling depends on registry lifecycle semantics
* VRF-based suites are not PQ-safe; lattice suites provide PQ migration path

---

## 11. IANA Considerations

This document requests:

* Media type registrations for `application/spe+cbor` and `application/spe+json`
* A registry for `suite_id` values
* A registry for `binding.mode` values

---

## Appendix A: Post-Quantum Lattice Addendum

Initial PQC suite set:

* `SPE-PQC-MLKEM768-MLDSA65-AES256GCM`
* `SPE-PQC-MLKEM768-SLHDSA-SHA2-128s-AES256GCM`

Hybrid migration is RECOMMENDED until PQC VRF constructions are standardized.

Reserved namespace for future PQC VRF suites:

* `SPE-PQVrf-*`

---

**End of draft-iaai-spet-transport-00**

