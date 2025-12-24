
<p align="center">
  <img src="./Unfakable-Data_Securing-AI-with-Enforced-Provenance.png" alt="IAAI: Unfakable Data: Securing AI with Enforced Provenance">
</p>

# Provenance Working Group

This working group focuses on data provenance, cryptographic verification, and secure architectures for AI systems.

## Table of Contents

### Overview & Architecture
- [IAAI Architecture Overview: Provenance and Coherence](./iaai_architecture_overview_provenance_and_coherence.md) - Presents the IAAI unified provenance architecture as three orthogonal standards (DPKR, SPET, CDPN) that make provenance verification a mandatory prerequisite to data access rather than optional metadata.

### Draft Standards

#### CDPN (Cryptographic Data Provenance Network)
- [Draft IAAI CDPN 00](./CDPN/draft_iaai_cdpn_00.md) - Defines the Coherence-Driven Provenance Network protocol implementing receiver-local quality feedback without enforcing subjective truth, allowing nodes to rate content quality and adapt behavior based on feedback.

#### DPKR (Distributed Public Key Registry)
- [Draft IAAI DPKR 00](./DPKR/draft_iaai_dpkr_00.md) - Specifies the Domain Provenance Key Registry mechanism for domains to publish, rotate, and retire cryptographic signing keys with explicit lifecycle management and historical verifiability.

#### SPET (Secure Provenance Exchange Transport)
- [Draft IAAI SPET Transport 00](./SPET/draft_iaai_spet_transport_00.md) - Defines the Secure Provenance Envelope transport container using deterministic CBOR encoding and multiple cryptographic suites including VRF-based verifiable deterministic key derivation and post-quantum lattice modes.

### Transactions (Discussion Documents)
- [Beyond Blockchain: Four Surprising Truths About Digital Identity and Trust](./Transactions/Beyond%20Blockchain_%20Four%20Surprising%20Truths%20About%20Digital%20Identity%20and%20Trust.md) - Explains four cryptographic paradigms for provenance including public key verification, VDKD identity weaving, spam-proof networks, and VRF-based encryption key derivation, shifting from attached labels to inherent mathematical proof.
- [Briefing on Cryptographic Provenance and Secure Data Architectures](./Transactions/Briefing-on-Cryptographic-Provenance-and-Secure-Data-Architectures.md) - Synthesizes enforced provenance verification through hybrid signcryption envelopes and VDKD, with applications to securing RAG systems and enabling inter-organizational data exchange via coherence-driven networks.
- [Executive Summary: A Unified Architecture for Enforced Provenance](./Transactions/Executive-Summary_A-Unified-Architecture-for-Enforced-Provenance.md) - Describes a three-layer architecture combining VDKD watermarking, C2PA manifests, and hybrid signcryption wrapper to create "unfakeable" data inseparably linked to creator identity.
- [Lattice-Lock: The Roadmap to Quantum-Resistant Provenance Architecture](./Transactions/Lattice-Lock_The-Roadmap-to-Quantum-Resistant-Provenance-Architecture.md) - Outlines the quantum-safe migration strategy moving from vulnerable RSA/ECC to lattice-based cryptography while maintaining AES-256's quantum resistance for symmetric encryption.
- [Provenance with Public Key Encryption](./Transactions/Provenance-with-Public-Key-Encryption.md) - Exhaustive technical analysis of "encrypting with a private key" (digital signature with message recovery), explaining mathematical foundations, limitations, and practical implementation via hybrid signcryption envelopes.
- [RAG Vector Provenance: Secure Chunking Architecture](./Transactions/RAG-Vector-Provenance_Secure-Chunking-Architecture.md) - Details the "Blind Retrieval" RAG pipeline where vector databases store encrypted SPEVR envelopes alongside unencrypted embeddings, requiring AI agents to cryptographically verify and decrypt chunks before use.
- [RSA vs Database Encryption at Rest](./Transactions/RSA-vs-Database-Encryption-at-Rest.md) - Explains why databases use AES rather than RSA for storage, while RSA protects keys in a DEK/KEK hierarchy and increasingly secures provenance in immutable ledgers and vector databases.
- [System Design: Cryptographically Verifiable Provenance for RAG Pipelines](./Transactions/System-Design_Cryptographically-Verifiable-Provenance-for-RAG-Pipelines.md) - Specifies the complete SPEVR standard (version 2.0) with three cryptographic suites, data model structure, and end-to-end workflow ensuring data poisoning immunity through enforced verification.
- [The Living Network: How Smart Systems Share Healthy Data](./Transactions/The-Living-Network_How-Smart-Systems-Share-Healthy-Data.md) - Models inter-organizational data exchange as biological cells with security membrane, metabolism (coherence evaluation), and trusted knowledge base to create spam-immune networks without requiring consensus on truth.

---

[Back to Main README](../../../README.md)
