# Internet-Draft: Domain Provenance Key Registry (DPKR)

**Intended status:** Standards Track  
**Expires:** 2026-06-23  
**Version:** draft-iaai-dpkr-00

---

## Abstract

This document defines the **Domain Provenance Key Registry (DPKR)**, a domain-controlled mechanism by which an Internet domain publishes, authorizes, rotates, and retires **provenance signing keys** used with the Secure Provenance Envelope Transport (SPET).

DPKR establishes a **deterministic trust anchor** answering the question *“could this message be from that domain?”* prior to any cryptographic proof verification or content decryption. It enables receivers to mechanically determine domain membership, key validity, allowed cryptographic suites, and key lifecycle state, forming the first-stage “membrane” in a provenance-based information network.

DPKR is transport- and PKI-agnostic. This document specifies the **registry data model, semantics, lifecycle rules, and verification requirements**, while allowing multiple publication and discovery profiles (e.g., pinned bilateral, HTTPS, DNSSEC).

---

## Status of This Memo

This Internet-Draft is submitted in full conformance with the provisions of BCP 78 and BCP 79. Distribution of this memo is unlimited.

---

## 1. Terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in RFC 2119 and RFC 8174.

* **Domain Registry**: The authoritative publication of provenance keys and policy for a domain.
* **Issuer Domain**: DNS domain asserting provenance authority.
* **Provenance Key**: A public key authorized to generate provenance proofs (VRF, signatures, or hybrid constructs).
* **issuer_kid**: Key identifier scoped to an issuer domain.
* **Receiver Domain**: Domain consuming provenance envelopes.
* **Key Epoch**: A bounded time interval during which a key is valid for new signing.

---

## 2. Scope and Design Principles

### 2.1 Scope

DPKR defines:

* The **logical registry data model** for domain provenance keys
* **Key lifecycle states** and rotation semantics
* **Policy constraints** bound to keys (allowed suites, usage)
* **Verification requirements** for receivers consuming registry data

DPKR does **not** define:

* The transport protocol for provenance envelopes (see SPET)
* Semantic evaluation, coherence scoring, or feedback mechanisms
* A global public identity system

### 2.2 Design Principles

1. **Domain sovereignty**: Each domain is authoritative for its own provenance keys.
2. **Early rejection**: Registry checks occur before cryptographic proof verification.
3. **Algorithm agility**: Registries may authorize multiple cryptographic suites concurrently.
4. **Historical verifiability**: Past envelopes remain verifiable after key rotation.
5. **Explicit compromise semantics**: Key compromise does not retroactively erase history.

---

## 3. Registry Data Model (Logical)

Each domain registry publishes a **Registry Object** with the following logical structure:

```text
DomainRegistry := {
  domain: DNSName,
  registry_version: string,
  published_at: timestamp,
  keys: [ KeyEntry+ ],
  policy: DomainPolicy
}
```

### 3.1 KeyEntry

```text
KeyEntry := {
  kid: string,
  public_key: PublicKey,
  key_type: enum,
  allowed_suites: [ suite_id+ ],
  status: enum,
  valid_from: timestamp,
  valid_to: timestamp?,
  compromised_at: timestamp?,
  metadata: map?
}
```

#### 3.1.1 `kid`

* MUST be unique within the domain
* MUST be stable for the lifetime of the key
* MUST NOT be reused across unrelated keys

#### 3.1.2 `public_key`

* Encoded in a suite-appropriate format
* MAY be represented directly or via reference (profile-dependent)

#### 3.1.3 `key_type`

One of:

* `vrf`
* `signature`
* `hybrid`

#### 3.1.4 `allowed_suites`

* Enumerates cryptographic suites this key is authorized to use
* Receivers MUST reject envelopes whose `suite_id` is not listed

#### 3.1.5 `status`

One of:

* `active` — MAY be used for new envelopes
* `retired` — MUST NOT be used for new envelopes; MAY be used for verification
* `revoked` — MUST NOT be used; verification MAY be policy-restricted

#### 3.1.6 Time Fields

* `valid_from`: earliest acceptable signing time
* `valid_to`: optional end of signing authority
* `compromised_at`: time of known compromise (if applicable)

---

## 4. Domain Policy Object

```text
DomainPolicy := {
  default_suites: [ suite_id* ],
  require_hybrid: boolean?,
  max_key_lifetime: duration?,
  cache_ttl: duration?,
  clock_skew_tolerance: duration?
}
```

* Policies are **advisory but authoritative** for receivers enforcing stricter local rules.
* Receiver domains MAY apply stricter policies than those advertised.

---

## 5. Key Lifecycle Semantics

### 5.1 Rotation

* Domains SHOULD rotate active keys periodically.
* Rotation MUST introduce a new `kid`.
* Old keys SHOULD transition to `retired` status.

### 5.2 Historical Verification

* Registries MUST retain retired keys for a policy-defined retention period.
* Receivers MUST be able to verify envelopes created during a key’s valid epoch.

### 5.3 Compromise Handling

If a key is compromised:

* `status` MUST be set to `revoked`
* `compromised_at` MUST be populated

Receivers SHOULD apply the following policy:

* Envelopes with `created_at < compromised_at` MAY be verified
* Envelopes with `created_at >= compromised_at` MUST be rejected

---

## 6. Receiver Processing Rules (Normative)

Upon receiving an envelope claiming `issuer_domain` and `issuer_kid`:

1. Resolve the domain registry using an approved discovery profile
2. Verify registry authenticity per profile rules
3. Locate `KeyEntry` matching `issuer_kid`
4. Enforce:
   * key existence
   * key status
   * time validity
   * allowed cryptographic suite
5. Only after successful registry validation MAY cryptographic proof verification proceed

Failure at any step MUST result in rejection.

---

## 7. Discovery and Publication Profiles

DPKR supports multiple discovery profiles. At least one MUST be supported by compliant implementations.

### 7.1 Profile P1 — Bilateral Pinned Registry (RECOMMENDED for opt-in networks)

* Registry endpoint and verification key exchanged during onboarding
* Registry updates signed by pinned registry key
* Suitable for consortia and early deployments

### 7.2 Profile P2 — HTTPS Well-Known Registry

* Registry published at:
  `https://<domain>/.well-known/dpkr.json`
* MUST be authenticated via TLS
* Registry object MUST be signed

### 7.3 Profile P3 — DNSSEC / DANE (OPTIONAL)

* Registry hash or key reference published via DNSSEC
* Enables fast, cacheable verification

Profiles MAY be combined.

---

## 8. Caching and Availability

* Receivers MAY cache registry objects up to `cache_ttl`
* If registry is unreachable:
  * Cached registry MAY be used if not expired
  * Otherwise, envelopes SHOULD be rejected (fail-closed)

---

## 9. Security Considerations

* Registry authenticity is critical; compromise undermines provenance
* Key reuse across domains MUST NOT occur
* Receivers SHOULD log registry state used for envelope verification
* Clock skew tolerance MUST be explicitly configured

---

## 10. IANA Considerations

This document requests no immediate IANA actions.

Future documents MAY define registries for:

* Key status values
* Registry discovery profiles

---

## 11. Relationship to SPET

DPKR is a **required dependency** for Secure Provenance Envelope Transport (SPET).

SPET defines *how provenance is proven*; DPKR defines *who is allowed to prove it*.

---

**End of draft-iaai-dpkr-00**

