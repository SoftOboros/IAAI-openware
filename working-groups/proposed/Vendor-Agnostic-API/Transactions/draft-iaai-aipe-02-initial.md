Vendor-Agnostic API Working Group                    L. T. Editor Internet-Draft                                      December 23, 2025 Intended status: Standards Track Expires: June 23, 2026 Obsoletes: draft-iaai-aipe-02  
        Internet-Draft: AI API Provenance Extension (AIPE)  
                       draft-iaai-aipe-02

#### Abstract

This document defines the second revision of the AI API Provenance Extension (AIPE). It summarizes the key motivations and architectural changes that address the significant complexity and interoperability risks identified during the review of draft-iaai-aipe-01.This revision addresses extensive architectural and operational feedback by transitioning from the flexible but complex "tiered model" to a simplified, interoperable "profile-based model." The core purpose of this draft is to reduce implementation complexity, mitigate the risk of ecosystem fragmentation, and provide clear, prescriptive guidance on the appropriate use of high-security encryption features, which carry significant performance and operational costs. Furthermore, this draft hardens the security posture of the specification by normatively resolving the Server-Side Request Forgery (SSRF) risks associated with external payload references.

#### Summary of Changes from draft-01

This section provides a high-level overview for reviewers, contrasting the key architectural concepts of draft-01 with the simplifications and refinements introduced in this draft. The changes reflect a strategic shift from offering a flexible framework to prescribing interoperable profiles that prioritize security, performance, and adoptability.| **Area** | **draft-iaai-aipe-01**  **(The Framework)** | **draft-iaai-aipe-02**  **(The Profiles)** | **Rationale for Change** || \------ | \------ | \------ | \------ || **Core Model** | Tiered Matrix (5.1-5.4) offering à la carte feature selection. | Compliance Profiles (Public, Confidential) bundling prescriptive choices. | Reduce the "Exploded Matrix" of choices, which created high complexity and fragmentation risk. || **Tier 5.4 (Encryption)** | Presented as a high-security option alongside signature-based tiers. | Demoted to a specialized profile with explicit performance and cost warnings. | Address persistent viability concerns regarding latency and key rotation costs; prevent misuse for general provenance. || **Model Attestation** | Signature (default) or VRF for general attestation. | Signature-only for general attestation; VRF is an internal component of Tier 5.4. | Simplify choices for implementers and resolve the "Root of Trust" ambiguity by clarifying the claim is accountability, not execution proof. || **External Payloads (**  **ref\_uri**  **)** | Permitted with security constraints as part of the core specification. | Client-side resolution is normative; server-side fetching is a discouraged optional extension. | Eliminate the risk of Server-Side Request Forgery (SSRF) from the core specification by default. || **Complexity** | High (4 Tiers x 4 Key Profiles x 4 Privacy Profiles). | Low (2-3 prescriptive profiles bundling choices). | Improve adoptability and ensure interoperability by guiding implementers toward standard configurations. |

#### Conventions and Terminology

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in BCP 14 RFC2119 RFC8174 when, and only when, they appear in all capitals, as shown here.

#### 1\. Introduction: From Framework to Interoperable Profiles

This section details the evolution in architectural thinking from the previous draft and establishes the primary objective of draft-iaai-aipe-02: to ensure the standard is practically adoptable and promotes a secure, interoperable ecosystem.

##### 1.1 The Success and Challenge of draft-01

The previous draft, draft-iaai-aipe-01, successfully established the critical architectural separation between  **authenticity**  (origin and integrity verification via signatures) and  **confidentiality**  (access control via encryption). This modularization into distinct tiers was a significant improvement over the initial "encryption-first" approach.However, this flexibility introduced a new and significant challenge: operational complexity. Implementers were faced with an "Exploded Matrix" of choices across four tiers, multiple key discovery methods, and several privacy modes. This created a substantial risk of non-interoperable subsets, where one provider might support Tier 5.1 with JWKS while another supports Tier 5.4 with DPKR, preventing seamless integration and undermining the goal of a unified standard.

##### 1.2 The Goal of draft-02: Simplification and Prescription

The primary goal of this draft is to resolve the complexity of draft-01 and mitigate the risk of fragmentation. This is achieved by introducing a small number of  **Compliance Profiles** . These profiles are coherent, purpose-built packages that bundle specific tiers, key discovery methods, and privacy settings.This prescriptive approach guides implementers toward the "smallest standard that works" for their specific use case, directly addressing a core critique from the review process. By defining clear, secure-by-default configurations, this draft prioritizes interoperability and lowers the barrier to adoption.This document will now detail the core innovation of this revision: the Compliance Profiles.

#### 2\. Core Specification: Compliance Profiles

This section defines the Compliance Profiles that form the normative core of this specification. They are designed to replace the "à la carte" tier selection from the previous draft with clear, interoperable, and secure-by-default configurations that align with common industry use cases.

##### 2.1 Public Provenance Profile (RECOMMENDED BASELINE)

This is the default profile for the vast majority of use cases where authenticity and integrity are required, but confidentiality of the content itself is not. Examples include media provenance, verifiable citations in public-facing AI assistants, and performance-critical Retrieval-Augmented Generation (RAG) pipelines.An implementation of this profile  **MUST**  implement:

* **Tier 5.1 (Signed Context)** : Provides authenticity for input documents using detached digital signatures. Asynchronous verification is the default mode to minimize latency impact.  
* **Tier 5.2 (Attested Output)** : Provides provider accountability for generated responses, using the signature mode.  
* **Key Discovery** : The jwks profile for its widespread adoption and compatibility with existing OIDC infrastructure.  
* **Privacy Profile** : The full profile, ensuring maximum transparency for auditing and public verification.

##### 2.2 Confidential RAG Profile (SPECIALIZED)

This is a specialized profile intended for high-security environments where the primary threat model includes preventing unauthorized reading of data at rest within a shared, multi-tenant infrastructure. It is explicitly designed for use cases where data confidentiality within the RAG pipeline is a primary concern.An implementation of this profile  **MUST**  implement:

* **Tier 5.4 (Enforced Provenance)** : Ensures content cannot be read without successful verification by wrapping documents in encrypted envelopes. Verification  **MUST**  be synchronous.  
* **Tier 5.2 (Attested Output)** : Provides accountability for responses generated from confidential data.  
* **Key Discovery** : The dpkr profile is RECOMMENDED for its advanced key lifecycle features, such as compromise handling, which are critical in high-security contexts.  
* **Privacy Profile** : The minimal profile is RECOMMENDED to reduce metadata leakage.WARNING: The Confidential RAG Profile introduces significant performance latency and operational costs. Implementers must acknowledge and plan for these trade-offs. This profile is not suitable for general-purpose provenance. A detailed analysis of the performance, cost, and archival risks is provided in Section 3.The following section provides mandatory guidance for any implementation of the specialized encryption-based features of Tier 5.4.

#### 3\. Normative Guidance for Tier 5.4 (Enforced Provenance)

Due to persistent architectural concerns and the high operational cost of encryption-based provenance, this section provides mandatory warnings and guidance for any implementation of the  **Confidential RAG Profile** . Implementers of this profile must understand and accept these trade-offs.

##### 3.1 Explicit Threat Model

Tier 5.4 is architecturally sound  **only**  when the threat model includes preventing the unauthorized reading of data  *at rest*  within a shared, multi-tenant infrastructure where application-layer access controls are insufficient.It is the wrong tool for public media provenance, as it conflates secrecy with authenticity. Using encryption for public content adds computational overhead and archival risk without providing a meaningful security benefit over digital signatures.

##### 3.2 Performance and Cost Acknowledgment

To ensure informed adoption, implementations of this profile MUST feature these costs and trade-offs prominently in their public-facing technical documentation:

* **Latency:**  The decryption of RAG context documents introduces significant overhead. Based on performance analysis, this can add  **200-400ms of latency**  for retrieving and decrypting k=500 vectors, which can violate service-level agreements for real-time applications.  
* **Key Rotation Costs:**  Security best practices require periodic key rotation. In an encryption-based provenance system where keys are tied to data, rotation necessitates re-encrypting the entire dataset. For a 1-billion vector dataset, this can incur  **over $6,000 in KMS API fees for a single rotation event** . While hierarchical key management strategies, specified in draft-01, can mitigate this cost, they do not eliminate it.

##### 3.3 Archival Risk: The "Digital Dark Age"

Tier 5.4 introduces the risk of permanent data loss through a mechanism known as "Cryptographic Shredding," where deleting a key makes the associated data irrecoverable. This creates a critical long-term archival risk.

* The  **"Locked Box"**  model of Tier 5.4 means that if the cryptographic keys are lost due to administrative error, organizational failure, or technology obsolescence, the content becomes mathematically indistinguishable from random noise and is lost forever.  
* This contrasts sharply with the  **"Digital Vellum"**  model of Tier 5.1 (signatures), which is "fail-open." If the keys or certificates required to verify a signature are lost, the verification capability is lost, but the underlying content remains accessible, preserving the historical record.Next, this document clarifies the trust claims associated with model attestation.

#### 4\. Finalizing Model Attestation (Tier 5.2)

This section provides normative clarification for Model Attestation (Tier 5.2), resolving ambiguities from draft-01 by simplifying cryptographic options and defining the trust claim it provides.

##### 4.1 Deprecating VRF for General Attestation

To reduce complexity and align with widely deployed tooling, the mode: signature is now the only mechanism for general-purpose Tier 5.2 attestation. Standard digital signatures (e.g., ECDSA, EdDSA, RSA) are sufficient and RECOMMENDED.The mode: vrf\_vdkd is now considered an internal component of Tier 5.4's encrypted envelope verification process. It  **MUST NOT**  be used for general model attestation in other profiles, as its complexity provides no additional security benefit for this use case.

##### 4.2 Trust Claim: Accountability, Not Execution Proof

It is normatively stated that a Tier 5.2 attestation provides  **provider accountability** . It is a "cryptographic receipt"—a non-repudiable claim that the provider  *alleges*  to have used a specific model to generate a response. This is valuable for auditing, billing disputes, and contractual enforcement.However, a Tier 5.2 attestation  **does not**  constitute proof of actual model execution. Cryptographic proof that a specific model ran on specific hardware requires Trusted Execution Environments (TEEs) and hardware-backed attestation, which is out of scope for this specification.This document now addresses a critical security vulnerability from the previous draft.

#### 5\. Resolving the ref\_uri SSRF Vulnerability

This section specifies the normative handling of external payload references (ref\_uri) to eliminate the risk of Server-Side Request Forgery (SSRF) from the core specification.

##### 5.1 Client-Side Resolution as Normative Default

The RECOMMENDED architecture is client-side resolution. Clients  **SHOULD**  resolve all referenced URIs and submit the full payload inline within the request body. Provenance-aware endpoints  **MUST**  support receiving inline payloads to facilitate this secure pattern. This approach removes the SSRF risk from the API provider entirely.

##### 5.2 Server-Side Fetching as an Optional Extension

Server-side fetching of payloads via ref\_uri is no longer part of the core AIPE specification. Providers  **MAY**  choose to implement this functionality, but if they do, it  **MUST**  be advertised as a separate, optional extension.If this optional extension is implemented, it  **MUST**  adhere to the following strict security constraints as outlined in draft-01:

* Only the https:// scheme is permitted.  
* Resolution to private IP ranges (e.g., RFC 1918\) is forbidden.  
* Redirects must not be followed.  
* Strict limits on payload size must be enforced.The final section summarizes the key security trade-offs for implementers.

#### 6\. Security Considerations

This section summarizes the most critical security decisions an implementer must make when adopting AIPE, focusing on the trade-offs introduced by the new compliance profiles.

* **Profile Selection:**  Implementers should default to the  **Public Provenance Profile**  for all standard use cases. The  **Confidential RAG Profile**  should only be used if the explicit threat model of protecting data-at-rest in a multi-tenant environment is met and the associated performance, cost, and data-loss risks are acceptable.  
* **Attestation Trust:**  Implementers must treat Tier 5.2 attestations as a provider-signed receipt for audit and accountability purposes. They must not be interpreted as a guarantee of model execution, which requires out-of-scope hardware-level attestation.  
* **SSRF Risk:**  Implementers are strongly advised against implementing the optional server-side ref\_uri fetching extension. If it is deemed absolutely necessary, it must be deployed with robust, multi-layered network protections and treated as a significant expansion of the API's attack surface.

