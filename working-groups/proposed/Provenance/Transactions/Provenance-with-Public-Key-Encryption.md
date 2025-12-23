# **Cryptographic Provenance and Ownership: An Exhaustive Analysis of Reverse-Encryption Architectures and Public Key Infrastructure**

## **1\. Executive Summary**

The digital ecosystem is currently navigating a profound crisis of authenticity. As generative artificial intelligence and sophisticated editing tools democratize the ability to manipulate digital content, the necessity for robust, cryptographically verifiable provenance has never been more acute. The user’s proposed architectural mechanism—wherein data is "encrypted" locally with a private key, stored in this transformed state, and subsequently "decrypted" by a recipient using the associated public key—represents a distinct and rigorously definable approach to this problem. This report provides an exhaustive analysis of this specific mechanism, contextualizing it within the broader field of cryptography, information security, and decentralized identity systems.  
The core of the user’s request posits a system of **Enforced Provenance Verification**. By transforming the data such that the public key is mathematically required to revert it to plaintext, the system binds the data irrevocably to the creator. If the data is readable, it *must* have originated from the holder of the private key. This report identifies this mechanism technically as **Digital Signature with Message Recovery**, a cryptographic primitive distinct from the more common **Digital Signature with Appendix**. While conceptually elegant, this approach faces significant hurdles regarding data size limitations inherent to asymmetric moduli, often restricting direct application to payloads smaller than 256 bytes.  
To address the requirements of maintaining a provenance chain, ensuring ownership, and storing data alongside public keys, this document synthesizes a wide array of research. We explore the mathematical underpinnings of RSA reversibility, the limitations of padding schemes, and the "Signcryption" primitive. Furthermore, we analyze established frameworks that achieve these goals at scale, including the **Coalition for Content Provenance and Authenticity (C2PA)**, the **W3C PROV** data model, **Self-Certifying File Systems (SFS)**, and decentralized approaches utilizing **IPFS/IPLD** and **Ocean Protocol**. The report concludes that while the user's "reverse encryption" model is mathematically valid, practical implementation for arbitrary file sizes requires a hybrid architecture—encapsulating symmetric keys within the asymmetric signature block—to achieve the desired binding of storage, identity, and access.

## **2\. Theoretical Foundations of Asymmetric Provenance**

To rigorously evaluate the proposal of "encrypting with a private key," one must first deconstruct the mathematical and operational realities of Public Key Cryptography (PKC). The terminology used in the query reflects a specific conceptual model that, while intuitively sound, encounters significant hurdles in practical implementation due to the design of modern cryptosystems and the mathematical properties of trapdoor functions.

### **2.1 The Mathematical Duality of RSA Keys**

The RSA algorithm, introduced by Rivest, Shamir, and Adleman in 1977 1, is the most prominent example of a cryptosystem where the user's proposed mechanism is mathematically viable. Unlike Diffie-Hellman or Elliptic Curve Cryptography (ECC), which are primarily key exchange or signature algorithms, RSA relies on the computational difficulty of factoring the product of two large prime numbers ($p$ and $q$) and operates through modular exponentiation that is theoretically reversible.1  
The generation of keys in RSA reveals why the user's proposal is possible:

1. **Prime Selection:** Choose two distinct large prime numbers $p$ and $q$.  
2. **Modulus Computation:** Compute $n \= pq$. This $n$ is used as the modulus for both public and private keys and determines the maximum message size.3  
3. **Totient Function:** Compute Euler's totient function $\\phi(n) \= (p-1)(q-1)$ (or Carmichael's totient function $\\lambda(n)$).1  
4. **Public Exponent:** Choose an integer $e$ such that $1 \< e \< \\phi(n)$ and $gcd(e, \\phi(n)) \= 1$. The pair $(n, e)$ is the **Public Key**.  
5. **Private Exponent:** Determine $d$ as $d \\equiv e^{-1} \\pmod{\\phi(n)}$. The pair $(n, d)$ is the **Private Key**.1

In "Textbook RSA," encryption and decryption are inverse operations governed by the relation $(M^e)^d \\equiv M \\pmod n$. Because exponentiation is commutative in this group, the inverse operation is equally valid:

$$(M^d)^e \\equiv M \\pmod n$$  
This second equation represents the exact workflow requested:

* **"Encrypt with Private Key":** The user applies $d$ to the data $M$ to create a ciphertext $S \= M^d \\pmod n$.  
* **"Decrypt with Public Key":** The recipient applies $e$ to $S$ to recover $M \= S^e \\pmod n$.5

In this context, the successful "decryption" of the data proves that it must have been transformed by the holder of the private key $d$, thereby establishing **provenance**. If the data decrypts to garbage, the provenance check fails. This satisfies the user's requirement that the act of decryption serves as the verification of ownership.6

### **2.2 Terminology: Encryption vs. Signatures**

In the strict lexicon of professional cryptography, the term "encryption" is reserved exclusively for processes that provide **confidentiality**.7

* **Confidentiality:** Ensuring that only authorized parties can read the data.  
* **Authenticity:** Ensuring the data originated from a specific source.

If a message is transformed using a private key, and the corresponding public key is widely available (as implied by the term "public"), then the message is not confidential. Anyone with the public key can "decrypt" and read it. Therefore, this operation is technically classified not as encryption, but as **Digital Signing**.  
However, the user's request distinguishes itself from standard signing by requiring the data to be *stored* in this transformed state. Standard signatures typically append a tag to the plaintext (Signature with Appendix). The user's model transforms the plaintext itself into the signature. This specific primitive is known in cryptography as **Digital Signature with Message Recovery**.9  
Table 1: Comparison of Cryptographic Primitives

| Feature | Encryption (Standard) | Signature (Appendix) | Signature (Message Recovery) |
| :---- | :---- | :---- | :---- |
| **Primary Goal** | Confidentiality (Secrecy) | Authenticity (Integrity) | Authenticity \+ Storage Efficiency |
| **Key Used to Transform** | Recipient's Public Key | Sender's Private Key | Sender's Private Key |
| **Key Used to Revert** | Recipient's Private Key | Sender's Public Key (Verify) | Sender's Public Key (Recover) |
| **Input** | Plaintext Message | Hash of Message | Plaintext Message (Small) |
| **Output** | Ciphertext | Plaintext \+ Signature Tag | "Ciphertext" (Recoverable Sig) |
| **Visibility** | Opaque without PrivKey | Visible (Plaintext sent) | Opaque without PubKey |
| **User's Request?** | No | No | **Yes** |

### **2.3 The Modulus Constraint and Data Size**

A critical and often overlooked limitation of the user's proposed mechanism ("encrypt the data locally with a private key") is the **size of the data**. Asymmetric operations like RSA are mathematical functions performed modulo $n$. This means the input message $M$ cannot be larger than the modulus $n$.11  
If a user wishes to "encrypt" a 5MB image using a 2048-bit RSA key, they face an immediate mathematical barrier.

* **Key Size:** A typical RSA key is 2048 bits, which equates to 256 bytes.  
* **Padding Overhead:** To be secure, RSA requires padding (like PKCS\#1 v1.5 or OAEP). This padding consumes bytes within the modulus. For example, PKCS\#1 v1.5 padding requires at least 11 bytes of overhead.11  
* **Maximum Payload:** With a 2048-bit key and standard padding, the maximum amount of data that can be "encrypted" (signed with recovery) in a single operation is $256 \- 11 \= 245$ bytes.11

Consequently, one cannot simply "encrypt" a large file with a private key in a single pass. To implement the user's request for arbitrary data types (images, documents), the system would have to split the file into 245-byte blocks and perform a computationally expensive RSA private key operation on *each* block.5 This approach, known as "Electronic Codebook (ECB) mode" for asymmetric keys, is computationally prohibitive (asymmetric crypto is approximately 1,000 times slower than symmetric crypto) and significantly expands the file size.13  
**Insight:** The user's desire to store data "encrypted" with the private key is feasible only for very small data packets (e.g., metadata, cryptographic keys, or small text assertions). For larger content, the architecture must adapt: the "data" stored in the provenance chain is usually a **hash** or a **symmetric key** that protects the actual payload. This necessitates a "Hybrid Signcryption" approach.

## **3\. Detailed Architecture of "Private Key Encryption" for Provenance**

The user's model—storing data as a blob that requires the public key to access—creates a system of **Enforced Provenance Verification**. Unlike a standard signature (where a user might ignore the signature file and just read the content), this model forces the user to perform the cryptographic verification operation to access the content at all. This has distinct architectural advantages for "maintaining user provenance" as requested.

### **3.1 Mechanism of Action: The Hybrid Envelope**

To overcome the size limitations described in Section 2.3 while satisfying the user's requirement for "encrypting locally," the standard practice is to use a **Hybrid Cryptosystem**.

#### **3.1.1 Step 1: Symmetric Encryption of Content**

The data ($D$), which may be large (e.g., high-resolution media), is first encrypted using a symmetric algorithm like AES-256 (Advanced Encryption Standard). Symmetric algorithms are fast and do not have the same size constraints as RSA.

* **Action:** $C\_{data} \= AES\\\_Encrypt(D, K\_{sym})$  
* Here, $K\_{sym}$ is a randomly generated symmetric session key.

#### **3.1.2 Step 2: Asymmetric "Encryption" (Signcryption) of the Key**

The symmetric key $K\_{sym}$ (typically 32 bytes for AES-256) is small enough to fit easily inside a single RSA block (245 bytes). The user's Private Key ($K\_{priv}$) is used to "encrypt" (sign with recovery) this session key.

* **Action:** $K\_{enc} \= RSA\\\_Sign\\\_Recover(K\_{sym}, K\_{priv})$  
* This creates a cryptographic blob $K\_{enc}$ that contains the session key.

#### **3.1.3 Step 3: Storage and Provenance Binding**

The system stores the bundle $\\{C\_{data}, K\_{enc}, K\_{pub}\\}$.

* **$C\_{data}$:** The opaque, encrypted data.  
* **$K\_{enc}$:** The "key to the door," which is locked by the creator's identity.  
* **$K\_{pub}$:** The public key provided for "decryption."

#### **3.1.4 Step 4: Decryption and Verification**

When a third party wishes to access the data:

1. They take $K\_{enc}$ and apply the provided $K\_{pub}$.  
2. **Operation:** $K\_{recovered} \= RSA\\\_Verify\\\_Recover(K\_{enc}, K\_{pub})$.  
3. If this operation succeeds, two things happen simultaneously:  
   * **Provenance Verified:** The mathematics confirm that only $K\_{priv}$ could have created $K\_{enc}$.  
   * **Access Granted:** The user now possesses $K\_{sym}$.  
4. The user decrypts $C\_{data}$ using $K\_{sym}$ to view the original content $D$.

This hybrid architecture strictly satisfies the user's requirement: the data is stored as encrypted, and the public key is required to decrypt it, thereby maintaining the provenance chain.5

### **3.2 Security Implications and Best Practices**

The query asks to research if this is "common practice" and identifying potential pitfalls. While the "encrypt with private key" mental model is useful, direct implementation has nuances that can lead to "bad practice" labels in security audits if not handled correctly.

#### **3.2.1 The "Bad Practice" Debate**

Security professionals often advise against "encrypting with a private key".14

1. **Confidentiality Confusion:** Users might believe the data is secret. It is not. Since the public key is public, anyone can decrypt it. It is merely **obfuscated**. The system must be labeled as a **Provenance System**, not a privacy system.  
2. **Key Separation:** Standard practice dictates using different key pairs for different purposes. A key used for *Signing* (Identity) should not be used for *Encryption* (receiving secret messages).15  
   * *Risk:* If the same key is used for both, an attacker might trick a user into "decrypting" a message that is actually a "signature" on a malicious file.  
   * *Mitigation:* Use a dedicated key pair strictly for this provenance workflow.

#### **3.2.2 Padding Oracle Vulnerabilities**

Using the wrong padding scheme when performing reverse operations can introduce vulnerabilities. Standard encryption padding (PKCS\#1 v1.5 type 2\) is different from signature padding (PKCS\#1 v1.5 type 1 or PSS).

* **Requirement:** When "encrypting with a private key," one must explicitly use **Signature Padding** modes (like PSS \- Probabilistic Signature Scheme) even though the intent is to wrap data. Using encryption padding in reverse can leak information about the private key.16

### **3.3 Signcryption Primitives**

The academic term for the simultaneous achievement of confidentiality and authenticity is **Signcryption**.18

* **Traditional Signcryption:** Usually provides confidentiality (only one receiver) and authenticity (sender verification).  
* **Publicly Verifiable Signcryption (PVS):** This is the exact primitive the user describes. It allows *anyone* (the public) to verify the ciphertext's origin.20  
  * In PVS, the ciphertext $C$ contains enough information for any third party to verify the sender's signature without necessarily needing a secret key, or by using the sender's public key alone.

## **4\. Industry Standards for Provenance Chains**

While the user's proposed mechanism is valid, the industry has converged on specific standards to maintain provenance chains at scale. These standards accomplish the user's goals—binding data to identity—often using similar underlying mathematics but different structural implementations.

### **4.1 The Coalition for Content Provenance and Authenticity (C2PA)**

The C2PA standard 22 is the most sophisticated implementation of public/private key provenance currently in use (adopted by Adobe, Microsoft, Intel, BBC). It aligns closely with the user's desire to bind data to an identity, though it typically leaves the data unencrypted (viewable) and attaches the provenance as a layer.

#### **4.1.1 The C2PA Manifest Structure**

C2PA does not "encrypt" the asset to hide it; instead, it cryptographically binds **Manifests** to the asset.

* **Asset:** The image, video, or document.  
* **Assertions:** Statements about the asset (e.g., "Created by Alice," "Edited with Photoshop," "Captured at Location X").  
* **Claim:** A set of assertions hashed together.  
* **Claim Signature:** The hash of the Claim is signed with the Creator's **Private Key**.

#### **4.1.2 Hard Binding vs. Soft Binding**

C2PA supports two methods of associating the provenance data with the file 22:

1. **Embedded (Hard Binding):** The manifest and signature are written directly into the file's metadata headers (e.g., JPEG markers, MP4 boxes). This ensures the provenance travels with the file, satisfying the "stored alongside" requirement.  
2. **Sidecar/Cloud (Soft Binding):** The file contains a reference (a hash or URL) to a manifest stored externally.

#### **4.1.3 The Trust Model**

The C2PA model validates provenance by checking the **Trust Chain** of the public key used to verify the signature.22

* The User provides the Public Key (via a Certificate).  
* The Viewer verifies the signature.  
* The Viewer checks if the Public Key is issued by a trusted Certificate Authority (CA) or is on a "Trust List."

**Relevance to User Query:** C2PA achieves the "provenance chain" requirement robustly. However, it *does not* strictly require the user to "decrypt" the data to see it. The data is usually visible, and the provenance is a verifiable overlay. If the user *must* enforce that the data is unreadable without the public key, C2PA manifests would need to be wrapped in the Hybrid Signcryption envelope described in Section 3.1.

### **4.2 W3C PROV: The Semantic Data Model**

While C2PA handles the cryptography of media, **W3C PROV** 24 provides the vocabulary to describe the *history*. It is a data model, not a cryptographic protocol, but it is essential for the "Chain" aspect of provenance requested by the user.

#### **4.2.1 Core Entities and Relationships**

W3C PROV defines three core types that map to the cryptographic components:

* **Entity:** A physical or digital thing (The Document / The Ciphertext).  
* **Activity:** Something that occurs over time and acts upon entities (The Encryption Event / The Signing Event).  
* **Agent:** A person or software bearing responsibility (The User holding the Private Key).

#### **4.2.2 Constructing the Provenance Chain**

In W3C PROV, a chain is established through derivations 25:

1. Entity B (Ciphertext) **wasGeneratedBy** Activity (Encryption).  
2. Activity (Encryption) **wasAssociatedWith** Agent (User).  
3. Entity B **wasDerivedFrom** Entity A (Plaintext).

**Integration:** A comprehensive system would use W3C PROV syntax (JSON-LD or XML) *inside* the encrypted payload or the C2PA assertion to describe exactly what happened: "This data was encrypted by Agent X using Key Y at Time Z." This provides the semantic layer of the provenance chain, while the RSA signature provides the cryptographic enforcement.

## **5\. Decentralized and Self-Sovereign Provenance**

The user's request mentions maintaining "certain ownership" and storing data "alongside its public key." This points strongly towards decentralized storage models where data is content-addressed and ownership is cryptographic rather than administrative (like a file server login).

### **5.1 Self-Certifying File Systems (SFS)**

The **Self-Certifying File System (SFS)** 27 is a historical but highly relevant architecture that matches the user's description almost perfectly. SFS was designed to allow secure file access across administrative boundaries without a central Certificate Authority.

#### **5.1.1 The Self-Certifying Pathname**

In SFS, the *filename* (or directory name) is cryptographically derived from the **Public Key** of the server/owner.

* **Format:** /sfs/\<Location\>:\<HostID\>/path/to/file  
* **HostID:** Hash(Server\_Public\_Key)

#### **5.1.2 Mechanism of Verification**

When a client attempts to access /sfs/host:K/file, the server presents its Public Key ($Pub\_S$).

1. **Verification:** The client computes $Hash(Pub\_S)$.  
2. **Comparison:** The client compares this hash to the $K$ in the pathname.  
3. **Connection:** If they match, the client knows it is talking to the correct server. The server then proves it possesses the Private Key ($Priv\_S$) by signing the connection handshake.

**Provenance Implication:** In SFS, the provenance is inherent in the link. You cannot spoof the storage location because the location identifier *is* the cryptographic identity. This fulfills the user's requirement of "Data stored... alongside its public key" (the key hash is part of the address).

### **5.2 IPFS and Content Addressing**

**IPFS (InterPlanetary File System)** 30 evolves the SFS concept using **CIDs (Content Identifiers)** and **IPNS (InterPlanetary Name System)**.

#### **5.2.1 CIDs: Integrity vs. Provenance**

* **CID:** A hash of the content itself. CID \= Hash(Data).  
* **Integrity:** If one bit of the data changes, the CID changes. This proves *what* the data is, but not *who* created it. A CID alone does not provide user provenance.

#### **5.2.2 IPNS: The Mutable Provenance Pointer**

To add provenance, IPFS uses IPNS. An IPNS name is the hash of a **Public Key**.

* **IPNS Name:** k51... (Hash of User's Public Key).  
* **Record:** A signed message (signed by User's Private Key) saying "My Name (k51...) points to this CID (Qm...)."  
* **Resolution:** When a user resolves an IPNS name, they fetch the signed record, verify the signature against the public key, and then retrieve the data at the CID.

**Insight:** This is the modern Web3 equivalent of the user's request. The data is stored publicly (on IPFS nodes), but its provenance is maintained because it is referenced by a record signed by the user's private key. The public key (hashed) serves as the persistent identity of the owner.

### **5.3 Ocean Protocol: Tokenized Ownership**

**Ocean Protocol** 32 takes ownership a step further by "tokenizing" the access control, implementing the "maintain certain ownership" requirement via blockchain assets.

#### **5.3.1 Data NFTs and Datatokens**

* **Data NFTs (ERC-721):** Represent the *Copyright/Ownership* of the dataset. The NFT is held by the wallet (Private Key owner) that deployed the data service.35  
* **Datatokens (ERC-20):** Represent the *License/Access* to the dataset.  
* **Encrypted DDO:** The Metadata (DDO \- DID Document) describes the data. Sensitive fields (like the direct URL to the file) are encrypted. Only the Ocean Provider (acting for the owner) can decrypt these fields using the private key logic.

#### **5.3.2 Compute-to-Data**

Ocean introduces **Compute-to-Data** 32, where the data never leaves the owner's control. Instead of the user "decrypting" the file locally, they send an algorithm to the owner's infrastructure. The owner's infrastructure (verified by private key) runs the algorithm and returns the result. This maintains maximum ownership while allowing usage.

## **6\. Comparison of Provenance Architectures**

To assist in selecting the optimal implementation, Table 2 contrasts the user's proposed "Reverse Encryption" method with the standard architectures identified in the research.  
Table 2: Architectural Comparison for Provenance Chains

| Feature | User Proposal (Reverse Encryption) | C2PA (Manifests) | IPFS \+ IPNS (Decentralized) | Ocean Protocol (Tokenized) | PKCS\#7 (SignedData) |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Mechanics** | Data \= $M^d \\pmod n$ | Data \+ Signed Metadata | Signed Pointer to Data Hash | NFT \+ Encrypted Metadata | Container (Data \+ Signature) |
| **Data Visibility** | **Opaque** (Requires PubKey) | **Transparent** (Visual) | Transparent (at CID) | Controlled (Gated) | **Opaque** (Encapsulated) |
| **Size Limit** | Severe (\< 245 bytes) | Unlimited | Unlimited | Unlimited | Unlimited |
| **Binding** | Mathematical (Inseparable) | Cryptographic (Linked) | Logical (Pointer) | Ledger (Smart Contract) | Structural (Wrapped) |
| **Provenance** | **High** (Key-derived) | **High** (Cert Chain) | **High** (Signed Record) | **High** (On-chain) | **High** (Embedded Cert) |
| **Standard?** | Non-Standard (Custom) | **Media Industry Std** | **Web3 Std** | **Data Market Std** | **Document Std** |

## **7\. Strategic Recommendations for Implementation**

Based on the synthesis of the user's "Encrypt with Private Key" requirement and the capabilities of modern systems, the following architectural patterns are recommended to build the requested solution.

### **7.1 Recommendation A: The "Self-Extracting" Provenance Blob (PKCS\#7/CMS)**

If the user prioritizes a standardized file format that contains both the data and the provenance key, the best fit is **PKCS\#7 / CMS (RFC 5652\)** using the SignedData content type.36  
**Implementation:**

* **Encapsulation:** The raw data (file) is placed inside the EncapsulatedContentInfo field of the CMS structure.  
* **Signing:** The data is hashed, and the hash is signed with the Creator's Private Key.  
* **Key Storage:** The Creator's Public Key (Certificate) is embedded in the certificates field of the blob.  
* **Result:** A single file (e.g., .p7m) that contains the data (opaque to casual viewers) and the key to verify it.  
* **User Workflow:** The recipient opens the blob. The software applies the public key to the signature. If valid, the data is extracted ("decrypted" from the structure).

### **7.2 Recommendation B: The "Hybrid Signcryption" Envelope**

To strictly implement the "encrypt with private key" logic for large files (satisfying the requirement that the public key is needed to decrypt):  
**Implementation:**

1. **Generate Session Key ($K\_{sym}$):** Create a random AES-256 key.  
2. **Encrypt Data:** $C\_{data} \= AES(Data, K\_{sym})$.  
3. **Signcrypt Key:** Use the Private Key to "Sign with Recovery" the $K\_{sym}$.  
   * $K\_{block} \= RSA\\\_Sign\\\_Recover(K\_{sym}, PrivKey)$.  
4. **Package:** Store $\\{K\_{block}, C\_{data}\\}$ alongside the Public Key.  
5. **Access:** The recipient *must* use the Public Key on $K\_{block}$ to recover $K\_{sym}$. Without the Public Key, they cannot get the symmetric key, and thus cannot read the data.

### **7.3 Recommendation C: The Decentralized "SFS-Style" Pointer**

For a web-scale system where ownership needs to be publicly verifiable without central servers:  
**Implementation:**

1. **Store Data:** Store the encrypted data on IPFS, generating a CID.  
2. **Create Identity:** Generate an IPNS record (Public Key Hash).  
3. **Bind:** Sign a record mapping IPNS\_Name \-\> CID.  
4. **Provenance:** The "Address" of the data is the User's Public Key. The data is stored logically "inside" the user's namespace.

## **8\. Conclusion**

The user's intuition—that data "encrypted" with a private key serves as the ultimate proof of provenance—is sound in principle and maps to the cryptographic primitive of **Digital Signature with Message Recovery**. However, the research indicates that direct implementation on large files is impractical due to modulus size constraints (typically \< 256 bytes).  
To fulfill the request of "maintaining user provenance" by requiring the public key for access, the report concludes that a **Hybrid Signcryption Architecture** is the optimal solution. This involves:

1. **Symmetric Encryption** of the payload for efficiency.  
2. **Private Key "Encryption" (Signing with Recovery)** of the symmetric key.  
3. **Bundled Storage** of the encrypted payload, the signcrypted key, and the public key certificate.

This architecture ensures that the data is inextricably bound to the creator's identity—accessing the content *mechanically requires* verifying the provenance chain—while avoiding the performance penalties and size limits of textbook RSA. By adopting this hybrid model, potentially wrapped in **PKCS\#7** or referenced via **IPNS**, the system achieves a robust, scalable, and secure provenance chain that strictly adheres to the user's ownership requirements.  
---

Citations:  
.1

#### **Works cited**

1. RSA cryptosystem \- Wikipedia, accessed December 23, 2025, [https://en.wikipedia.org/wiki/RSA\_cryptosystem](https://en.wikipedia.org/wiki/RSA_cryptosystem)  
2. RSA Algorithm in Cryptography \- GeeksforGeeks, accessed December 23, 2025, [https://www.geeksforgeeks.org/computer-networks/rsa-algorithm-cryptography/](https://www.geeksforgeeks.org/computer-networks/rsa-algorithm-cryptography/)  
3. RSA Encryption : Beginners guide. What the heck is Encryption anyway? | by Vishwas Adhikari | Medium, accessed December 23, 2025, [https://medium.com/@Espress0/rsa-encryption-beginners-guide-6caee3e8cd80](https://medium.com/@Espress0/rsa-encryption-beginners-guide-6caee3e8cd80)  
4. RSA (explained step by step) \- CrypTool, accessed December 23, 2025, [https://www.cryptool.org/en/cto/rsa-step-by-step/](https://www.cryptool.org/en/cto/rsa-step-by-step/)  
5. What is the difference between encrypting and signing in asymmetric encryption? \[closed\], accessed December 23, 2025, [https://stackoverflow.com/questions/454048/what-is-the-difference-between-encrypting-and-signing-in-asymmetric-encryption](https://stackoverflow.com/questions/454048/what-is-the-difference-between-encrypting-and-signing-in-asymmetric-encryption)  
6. How Digital Signatures Work: Types, Benefits, And More \- SSH Communications Security, accessed December 23, 2025, [https://www.ssh.com/academy/secure-information-sharing/how-digital-signatures-work](https://www.ssh.com/academy/secure-information-sharing/how-digital-signatures-work)  
7. Encryption and Digital Signatures | Information Security | University of Houston-Clear Lake, accessed December 23, 2025, [https://www.uhcl.edu/information-security/tips-best-practices/encryption](https://www.uhcl.edu/information-security/tips-best-practices/encryption)  
8. Digital certificates: What is the difference between encrypting and signing \- Stack Overflow, accessed December 23, 2025, [https://stackoverflow.com/questions/21823679/digital-certificates-what-is-the-difference-between-encrypting-and-signing](https://stackoverflow.com/questions/21823679/digital-certificates-what-is-the-difference-between-encrypting-and-signing)  
9. Message Recovery Signature Schemes from Sigma-protocols | NTT Technical Review, accessed December 23, 2025, [https://www.ntt-review.jp/archive/ntttechnical.php?contents=ntr200801sp2.html](https://www.ntt-review.jp/archive/ntttechnical.php?contents=ntr200801sp2.html)  
10. Signing with recovery \- Cryptsoft, accessed December 23, 2025, [https://www.cryptsoft.com/pkcs11doc/v210/group\_\_SEC\_\_13\_\_4\_\_SIGNING\_\_WITH\_\_RECOVERY.html](https://www.cryptsoft.com/pkcs11doc/v210/group__SEC__13__4__SIGNING__WITH__RECOVERY.html)  
11. RSA Encryption Maximum Number of Bytes \- Chilkat Tech Notes, accessed December 23, 2025, [https://cknotes.com/rsa-encryption-maximum-number-of-bytes/](https://cknotes.com/rsa-encryption-maximum-number-of-bytes/)  
12. how to calculate maximum data size of an RSA signature \- Cryptography Stack Exchange, accessed December 23, 2025, [https://crypto.stackexchange.com/questions/30826/how-to-calculate-maximum-data-size-of-an-rsa-signature](https://crypto.stackexchange.com/questions/30826/how-to-calculate-maximum-data-size-of-an-rsa-signature)  
13. RSA Algorithm in Cryptography: Rivest Shamir Adleman Explained | Splunk, accessed December 23, 2025, [https://www.splunk.com/en\_us/blog/learn/rsa-algorithm-cryptography.html](https://www.splunk.com/en_us/blog/learn/rsa-algorithm-cryptography.html)  
14. Is encrypting with private key instead of signing a bad idea? \- Stack Overflow, accessed December 23, 2025, [https://stackoverflow.com/questions/57535111/is-encrypting-with-private-key-instead-of-signing-a-bad-idea](https://stackoverflow.com/questions/57535111/is-encrypting-with-private-key-instead-of-signing-a-bad-idea)  
15. Why should one not use the same asymmetric key for encryption as they do for signing?, accessed December 23, 2025, [https://security.stackexchange.com/questions/1806/why-should-one-not-use-the-same-asymmetric-key-for-encryption-as-they-do-for-sig](https://security.stackexchange.com/questions/1806/why-should-one-not-use-the-same-asymmetric-key-for-encryption-as-they-do-for-sig)  
16. RSA — Cryptography 3.4.4 documentation, accessed December 23, 2025, [https://cryptography.io/en/3.4.4/hazmat/primitives/asymmetric/rsa.html](https://cryptography.io/en/3.4.4/hazmat/primitives/asymmetric/rsa.html)  
17. Is it more weak to encrypt with the private key RSA? \- Cryptography Stack Exchange, accessed December 23, 2025, [https://crypto.stackexchange.com/questions/76419/is-it-more-weak-to-encrypt-with-the-private-key-rsa](https://crypto.stackexchange.com/questions/76419/is-it-more-weak-to-encrypt-with-the-private-key-rsa)  
18. A secure data sharing scheme based on broadcast signcryption in data center \- PMC \- NIH, accessed December 23, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC12599950/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12599950/)  
19. Cloud Storage Data Verification Using Signcryption Scheme \- MDPI, accessed December 23, 2025, [https://www.mdpi.com/2076-3417/12/17/8602](https://www.mdpi.com/2076-3417/12/17/8602)  
20. A Directly Public Verifiable Signcryption Scheme based on Elliptic Curves \- ResearchGate, accessed December 23, 2025, [https://www.researchgate.net/publication/224576603\_A\_Directly\_Public\_Verifiable\_Signcryption\_Scheme\_based\_on\_Elliptic\_Curves](https://www.researchgate.net/publication/224576603_A_Directly_Public_Verifiable_Signcryption_Scheme_based_on_Elliptic_Curves)  
21. \[1107.1847\] Efficient Identity Based Public Verifiable Signcryption Scheme \- arXiv, accessed December 23, 2025, [https://arxiv.org/abs/1107.1847](https://arxiv.org/abs/1107.1847)  
22. Content Credentials : C2PA Technical Specification :: C2PA ..., accessed December 23, 2025, [https://spec.c2pa.org/specifications/specifications/1.0/specs/C2PA\_Specification.html](https://spec.c2pa.org/specifications/specifications/1.0/specs/C2PA_Specification.html)  
23. 1\. Introduction \- C2PA, accessed December 23, 2025, [https://c2pa.org/wp-content/uploads/sites/33/2025/10/content\_credentials\_wp\_0925.pdf](https://c2pa.org/wp-content/uploads/sites/33/2025/10/content_credentials_wp_0925.pdf)  
24. W3C Prov \- Wikipedia, accessed December 23, 2025, [https://en.wikipedia.org/wiki/W3C\_Prov](https://en.wikipedia.org/wiki/W3C_Prov)  
25. PROV-DM: The PROV Data Model \- W3C, accessed December 23, 2025, [https://www.w3.org/TR/prov-dm/](https://www.w3.org/TR/prov-dm/)  
26. PROV-DM: The PROV Data Model \- W3C, accessed December 23, 2025, [https://www.w3.org/2012/10/prov-dm](https://www.w3.org/2012/10/prov-dm)  
27. Self-certifying File System \- Wikipedia, accessed December 23, 2025, [https://en.wikipedia.org/wiki/Self-certifying\_File\_System](https://en.wikipedia.org/wiki/Self-certifying_File_System)  
28. Self-certifying File System \- USENIX, accessed December 23, 2025, [https://www.usenix.org/legacyurl/self-certifying-file-system](https://www.usenix.org/legacyurl/self-certifying-file-system)  
29. Using SFS for a Secure Network File System \- Stanford Secure Computer Systems Group, accessed December 23, 2025, [https://www.scs.stanford.edu/\~dm/home/papers/fu:using-sfs.pdf](https://www.scs.stanford.edu/~dm/home/papers/fu:using-sfs.pdf)  
30. IPLD InterPlanetary Ecosystem Overview, accessed December 23, 2025, [https://ipld.io/docs/intro/ecosystem/](https://ipld.io/docs/intro/ecosystem/)  
31. Glossary \- IPFS Docs, accessed December 23, 2025, [https://docs.ipfs.tech/concepts/glossary/](https://docs.ipfs.tech/concepts/glossary/)  
32. Meet Ocean: Tokenized AI & Data — Ocean Protocol, accessed December 23, 2025, [https://oceanprotocol.com/](https://oceanprotocol.com/)  
33. Ocean Protocol \- Quicknode, accessed December 23, 2025, [https://www.quicknode.com/builders-guide/tools/ocean-protocol-by-ocean-protocol-foundation](https://www.quicknode.com/builders-guide/tools/ocean-protocol-by-ocean-protocol-foundation)  
34. Ocean Protocol: Decentralized Data and Ocean Market \- Gemini, accessed December 23, 2025, [https://www.gemini.com/cryptopedia/ocean-protocol-web-3-0-ocean-market-ocean-token](https://www.gemini.com/cryptopedia/ocean-protocol-web-3-0-ocean-market-ocean-token)  
35. How Ocean Protocol is Powering Decentralized Data Sharing for AI, accessed December 23, 2025, [https://www.blockchainappfactory.com/blog/how-ocean-protocol-marketed-decentralized-data-for-ai/](https://www.blockchainappfactory.com/blog/how-ocean-protocol-marketed-decentralized-data-for-ai/)  
36. PKCS 7 \- Wikipedia, accessed December 23, 2025, [https://en.wikipedia.org/wiki/PKCS\_7](https://en.wikipedia.org/wiki/PKCS_7)  
37. Anatomy of CMS/PKCS \#7 signed messages \- JavaScience, accessed December 23, 2025, [https://www.jensign.com/sigview/index.html](https://www.jensign.com/sigview/index.html)  
38. Signature with appendix \- Cryptography Stack Exchange, accessed December 23, 2025, [https://crypto.stackexchange.com/questions/8820/signature-with-appendix](https://crypto.stackexchange.com/questions/8820/signature-with-appendix)  
39. C2PA Explainer :: C2PA Specifications, accessed December 23, 2025, [https://spec.c2pa.org/specifications/specifications/1.3/explainer/Explainer.html](https://spec.c2pa.org/specifications/specifications/1.3/explainer/Explainer.html)  
40. Content Authenticity Initiative \- Wikipedia, accessed December 23, 2025, [https://en.wikipedia.org/wiki/Content\_Authenticity\_Initiative](https://en.wikipedia.org/wiki/Content_Authenticity_Initiative)