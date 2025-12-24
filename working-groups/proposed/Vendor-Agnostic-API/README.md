<p align="center">
  <img src="./From-Locked-Box-To-Layered-Trust.png" alt="IAAI: /From Locked Box To Layered Trust.">
</p>

# Vendor-Agnostic API Working Group

This working group focuses on developing vendor-agnostic API standards for AI systems to enable interoperability and portability across different AI platforms and providers.

## Table of Contents

### **Overview & Foundational Documents**
- [The Locked Box vs. The Notary's Seal: A Tale of Two Ways to Trust AI](./Transactions/The%20Locked%20Box%20vs.%20The%20Notary's%20Seal_%20A%20Tale%20of%20Two%20Ways%20to%20Trust%20AI.md) - An accessible narrative explaining the fundamental debate between encryption-based (locked box) and signature-based (notary's seal) approaches to AI provenance, presenting the thesis, antithesis, and synthesis of the AIPE specification evolution.
- [Standardizing AI API Landscape](./Transactions/Standardizing-AI-API-Landscape.md) - Overview of the broader AI API standardization landscape, challenges, and opportunities for creating vendor-agnostic interfaces across diverse AI platforms.

### Draft Standards

#### AIPE (AI API Provenance Extension)
- [Draft IAAI AIPE 00](./AIPE/draft_iaai_aipe_00.md) - Original AIPE specification proposing "Level 5" compliance with encryption-based provenance verification, mandatory input verification, model attestation via VRF proofs, and quality indicator feedback mechanisms.
- [Draft IAAI AIPE 01](./AIPE/draft_iaai_aipe_01.md) - Major revision introducing a tiered provenance model separating signature-based provenance (Tiers 5.1-5.3) from encryption-based provenance (Tier 5.4), emphasizing that authenticity and confidentiality are orthogonal concerns.
- [Draft IAAI AIPE 02 (Initial)](./Transactions/draft-iaai-aipe-02-initial.md) - Second revision simplifying the tiered model into two prescriptive compliance profiles—Public Provenance (signature-based, RECOMMENDED) and Confidential RAG (encryption-based, SPECIALIZED)—while hardening security by normalizing client-side URI resolution.

### Transactions (Discussion Documents)
- [Rebutting AIPE Draft Proposal](./Transactions/Rebutting%20AIPE%20Draft%20Proposal.md) - Comprehensive technical rebuttal arguing AIPE's encryption-first approach conflates secrecy with authenticity, introduces catastrophic RAG latency penalties, and threatens long-term data preservation, recommending C2PA's signature-based approach instead.
- [Response to Socratic Rebuttal AIPE](./Transactions/Response_to_Socratic_Rebuttal_AIPE.md) - Measured response accepting several critiques and proposing concrete amendments including separating AIPE into sub-profiles, clarifying model attestation scope, and supporting multiple registry profiles.
- [Socratic Rebuttal AIPE](./Transactions/Socratic_Rebuttal_AIPE.md) - Question-driven stress test challenging whether AIPE is necessary, minimally sufficient, and operationally adoptable by interrogating scope creep, threat model clarity, and implementation feasibility.

### **Reference Materials**
- [Glossary of API Standardization Terms](./Transactions/Glossary%20of%20API%20Standardization%20Terms.md) - Foundational glossary defining key concepts in AI API standardization including the critical distinction between signature-based provenance (Digital Vellum—fail-open) and encryption-based provenance (Locked Box—fail-closed).

---

[Back to Main README](../../../README.md)
