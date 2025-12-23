The planned support for quantum-safe security within the provenance architecture is focused on transitioning from classical asymmetric algorithms to **Lattice-based cryptography** to ensure long-term viability against Shor’s algorithm 1-3.  
Here is a rehash of the quantum-safe support strategy based on the sources:

### 1\. The Current Quantum Vulnerability

The sources acknowledge that all currently recommended asymmetric suites—**Suite A (Ed25519)**, **Suite B (P-256)**, and **Suite C (RSA)**—are vulnerable to cryptanalysis by a sufficiently powerful quantum computer 2-4.

* **Shor's Algorithm:** This poses a "catastrophic" threat to the asymmetric layer because it can factor large integers and solve discrete logarithm problems, shattering the "Master Key" layer used for identity and key exchange 1, 5\.  
* **Suitability of RSA/ECC:** Because current **Verifiable Deterministic Key Derivation (VDKD)** relies on these classical curves and moduli, a quantum adversary could potentially derive the private key from a public key or proof 1, 2\.

### 2\. Quantum-Resistant Components (The "Workhorse")

While the asymmetric layer is at risk, the symmetric layer is already considered largely **quantum-resistant**.

* **AES-256 Strength:** The sources note that while **Grover’s algorithm** provides a quadratic speedup for brute-forcing symmetric keys, it only reduces the effective security of a 256-bit key to 128 bits 1, 6\.  
* **Practicality:** 128 bits of security is still considered a formidable barrier, meaning **AES-256** remains a safe "workhorse" for bulk data encryption at rest within the SPEVR and provenance envelopes 6, 7\.

### 3\. The Roadmap: Suite D and NTRU

To address the vulnerability of the "Identity Lock," the next iteration of the standard (**DRAFT-SPEVR-03**) is planned to introduce **Suite D** 2\.

* **Lattice-Based Primitives:** Suite D will utilize lattice-based cryptography, specifically **NTRU-based recoverable signatures** 8, 9\.  
* **Recoverable Signatures for VDKD:** Just as RSA-FDH is used today for "Signature with Message Recovery," NTRU-like algorithms will be used to embed the AES session key inside a quantum-resistant signature 9, 10\.  
* **Verification-Before-Access:** This ensures the "Enforced Provenance" model survives; a recipient must run a lattice-based verification algorithm to recover the decryption key, maintaining the mathematical requirement that access is identical to provenance verification 10, 11\.

### 4\. Strategic Outlook

The transition to post-quantum cryptography (PQC) is viewed as essential for "stewardship" and the "10-year test" of data integrity 3, 12\. By moving toward **Self-Certifying Identifiers (SCIDs)** and rotatable VRFs that can eventually support lattice-based proofs, the network aims to create a durable foundation where the "truth of origin" remains recoverable even in a post-quantum era 13-15.  
**Analogy for Understanding:**Think of the current system as a **high-security vault (AES-256)** that is locked with a **digital keypad (RSA/ECC)**. A "quantum crowbar" can easily bypass the keypad's logic to find the code. The quantum-safe plan isn't to replace the entire vault, but to replace the keypad with a **"Lattice-Lock"**—a mechanism so mathematically complex that the crowbar cannot find a point of leverage to force it open.  
