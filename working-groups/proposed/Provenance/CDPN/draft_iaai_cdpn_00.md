# Internet-Draft: Coherence-Driven Provenance Network (CDPN)

**Intended status:** Standards Track  
**Expires:** 2026-06-23  
**Version:** draft-iaai-cdpn-00

---

## Abstract

This document defines the **Coherence-Driven Provenance Network (CDPN)**: a protocol layer that governs **quality-aware, feedback-regulated information exchange** between domains participating in a provenance-secured transport (SPET) and identity authorization framework (DPKR).

CDPN specifies how receivers evaluate **coherence** of verified data artifacts, how **quality indicators** are generated and returned to senders, and how these indicators are intended to regulate ongoing information flow. The protocol deliberately avoids semantic “truth” enforcement and instead standardizes a **mechanical, opt-in, self-tuning feedback loop** that allows heterogeneous domains to exchange information without requiring shared beliefs, schemas, or objectives.

---

## Status of This Memo

This Internet-Draft is submitted in full conformance with the provisions of BCP 78 and BCP 79. Distribution of this memo is unlimited.

---

## 1. Terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in RFC 2119 and RFC 8174.

* **Node**: An autonomous domain participant implementing DPKR, SPET, and CDPN.
* **Sender Node**: Node originating a provenance envelope.
* **Receiver Node**: Node evaluating a received envelope.
* **Coherence Evaluation**: Receiver-local semantic assessment of a verified payload.
* **Quality Indicator (QI)**: A scalar feedback value communicated from receiver to sender.
* **Epistemic Engine**: Sender-local logic that adapts outbound behavior based on QI feedback.

---

## 2. Scope and Non-Goals

### 2.1 Scope

CDPN defines:

* Feedback message formats and semantics
* Required ordering between provenance verification and coherence evaluation
* The meaning and range of Quality Indicators
* Minimal expectations for sender-side adaptation

### 2.2 Non-Goals

CDPN explicitly does **not** define:

* How coherence is computed internally
* Any notion of objective truth or correctness
* Any requirement for shared ontologies, schemas, or models
* Any global reputation or scoring system

---

## 3. Architectural Position

CDPN operates **above** SPET and **assumes**:

1. The payload has passed **domain registry authorization** (DPKR)
2. The payload has passed **cryptographic provenance verification** (SPET)

Only after these conditions are met MAY coherence evaluation occur.

Failure at any lower layer MUST preclude CDPN processing.

---

## 4. Coherence Evaluation Model

### 4.1 Definition of Coherence

Coherence is defined as **receiver-local compatibility** between a verified payload and the receiver’s existing knowledge, objectives, and processing constraints.

Coherence evaluation MUST be:

* Deterministic with respect to receiver state at evaluation time
* Local to the receiver
* Non-authoritative beyond the sender–receiver relationship

### 4.2 Recommended Evaluation Dimensions (Informative)

Receivers MAY evaluate coherence using any internal method. The following dimensions are RECOMMENDED as a common interpretive basis:

1. **Relevance** – Alignment with receiver interests or subscribed topics
2. **Structure** – Well-formedness and parseability
3. **Non-Contradiction** – Absence of direct conflict with receiver-held invariants
4. **Novelty** – Degree of new information relative to receiver memory

No dimension is REQUIRED to be exposed or weighted uniformly.

---

## 5. Quality Indicator (QI)

### 5.1 Definition

A **Quality Indicator (QI)** is a scalar value in the closed interval **[0.0, 1.0]** representing the receiver’s aggregate assessment of coherence.

* `1.0` indicates maximal coherence
* `0.0` indicates incoherence or rejection

### 5.2 Threshold Semantics

Receivers MUST define a local **acceptance threshold** `T_accept`.

* If `QI >= T_accept`, the payload MAY be absorbed into receiver memory
* If `QI < T_accept`, the payload MUST NOT be absorbed

Threshold values are receiver-local and MAY vary by sender or topic.

---

## 6. Feedback Message

### 6.1 Feedback Generation

After coherence evaluation, the receiver MUST generate exactly one feedback outcome per processed envelope.

Feedback MUST be generated regardless of acceptance outcome.

### 6.2 Feedback Format

Feedback messages MUST include:

| Field | Description |
|---|---|
| `envelope_id` | Identifier of evaluated envelope |
| `receiver_domain` | Domain issuing feedback |
| `quality_indicator` | Scalar QI value |
| `evaluated_at` | Timestamp |

Optional fields MAY include:

* `reason_code` (enumerated, non-normative)
* `notes` (human-readable, non-normative)

Feedback messages SHOULD be transmitted using the same transport channel as SPET or a mutually agreed side-channel.

---

## 7. Sender-Side Adaptation (Epistemic Engine)

### 7.1 Required Sender Behavior

Senders implementing CDPN:

* MUST associate received QI values with the `(receiver_domain, topic/context)` pair
* MUST NOT treat QI as a global reputation score

### 7.2 Adaptive Expectations (Informative)

Senders are EXPECTED to:

* Reduce or cease transmission of low-QI payloads
* Adjust abstraction, granularity, or topic selection
* Prefer receivers yielding sustained high-QI feedback

No specific learning algorithm is mandated.

---

## 8. Flow Control and Abuse Resistance

### 8.1 Receiver Protections

Receivers MAY:

* Rate-limit CDPN processing per sender
* Suppress feedback for repeated incoherent submissions
* Escalate to registry-level blocking via local policy

### 8.2 Sender Abuse

Senders MUST NOT:

* Attempt to infer receiver internal state beyond QI
* Treat absence of feedback as positive signal

---

## 9. Privacy Considerations

QI values reveal minimal information but may leak:

* Receiver interest gradients
* Receiver acceptance thresholds

Receivers SHOULD avoid exposing detailed diagnostics.

---

## 10. Security Considerations

* CDPN assumes cryptographic authenticity via SPET
* Malicious feedback harms primarily the dishonest receiver
* Long-term dishonesty leads to pathway atrophy

CDPN relies on self-interest alignment rather than central enforcement.

---

## 11. Relationship to Other Specifications

* **DPKR** — determines *who may speak*
* **SPET** — determines *whether content is authentic and intact*
* **CDPN** — determines *whether content is useful*

These layers are intentionally orthogonal.

---

## 12. IANA Considerations

This document requests no immediate IANA actions.

Future documents MAY define:

* Standard `reason_code` enumerations
* Feedback transport bindings

---

**End of draft-iaai-cdpn-00**

