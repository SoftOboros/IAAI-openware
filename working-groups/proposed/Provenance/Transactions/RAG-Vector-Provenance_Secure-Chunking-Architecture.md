# **Secure RAG Provenance: Cryptographic Binding of Vector Origins**

## **1\. Executive Summary**

This report adapts the concept of "Enforced Provenance" (encrypting data with a private key) to the architecture of **Retrieval-Augmented Generation (RAG)** systems. The user's requirement is to ensure that every text chunk retrieved by an AI agent is cryptographically bound to its creator, such that the chunk cannot be read or used without "decrypting" it via the creator's public key.  
Applying this to **2KB chunks** introduces immediate cryptographic constraints: standard RSA keys (2048-bit) can only process \~245 bytes of data directly. Therefore, a **Hybrid Signcryption** pipeline is required. In this architecture, the RAG system performs "Blind Retrieval"—it searches using mathematical vectors (which are derived from the text but do not reveal it) and retrieves opaque, encrypted payloads. The consuming agent must then apply the public key to "unlock" the text, simultaneously verifying provenance and granting access.

## **2\. The Cryptographic Constraints: 2KB vs. 245 Bytes**

The user's defined constraint of a **2KB (2048 bytes)** chunk size presents a hard mathematical limit for direct asymmetric operations.

### **2.1 The Modulus Limit**

As established in the previous analysis, "encrypting with a private key" (mathematically $M^d \\pmod n$) is limited by the key size.

* **Key Size:** RSA-2048 allows a modulus of 256 bytes.  
* **Padding Overhead:** Safe padding (PKCS\#1 v1.5 or PSS) consumes \~11+ bytes.  
* **Max Payload:** $\\approx 245$ bytes.

### **2.2 The Implication for RAG**

A 2KB chunk is roughly $8\\times$ larger than the maximum capacity of a standard RSA operation. Breaking the chunk into 8 separate RSA blocks is computationally inefficient and complicates the vector retrieval process.

* **Solution:** We must utilize **Hybrid Signcryption**. The text chunk is encrypted symmetrically (AES), and the *symmetric key* is "encrypted" (signed with recovery) using the Private Key.

## **3\. Recommended Architecture: The "Blind Retrieval" RAG Pipeline**

This architecture ensures that the "Vector Origin Text" (the chunk stored in the database) is stored as an encrypted blob alongside its public key, satisfying the user's provenance requirement.

### **3.1 Phase 1: Ingestion & Chunking (Local Client)**

This phase must happen **locally** or within a trusted enclave to maintain user provenance. The private key never leaves this environment.

1. **Input:** Document (Max 1MB).  
2. **Chunking:** Split document into $N$ segments of max 2KB (approx. 500-1000 tokens).  
   * *Strategy:* Use **Semantic Chunking** or **Recursive Character Splitting** to ensure chunks are meaningful before encryption.  
3. **Vectorization (Plaintext):**  
   * Generate the embedding vector $V$ from the *plaintext* chunk.  
   * *Note:* The embedding model *must* see the plaintext to generate the vector. If this is a concern, a local embedding model (e.g., ONNX/quantized Llama) should be used.  
4. **Hybrid Provenance Binding (The "Redo" Step):**  
   * **Generate Key:** Create a random 32-byte AES Session Key ($K\_{sym}$).  
   * **Encrypt Chunk:** Encrypt the 2KB plaintext chunk using AES-256-GCM with $K\_{sym}$.  
     * Output: Encrypted\_Chunk ($C\_{data}$).  
   * **Signcrypt Key:** "Encrypt" $K\_{sym}$ using the User's RSA Private Key ($K\_{priv}$).  
     * Output: Provenance\_Token ($K\_{wrapped}$).  
5. **Payload Construction:**  
   * Create a JSON object containing:  
     * id: Unique Chunk ID.  
     * provenance\_token: Base64($K\_{wrapped}$).  
     * ciphertext: Base64($C\_{data}$).  
     * public\_key: The User's Public Key (or Certificate ID).

### **3.2 Phase 2: Storage (Vector Database)**

The Vector Database (e.g., Chroma, Pinecone, Milvus) stores the mathematical representation and the opaque blob. It *cannot* read the text.

| Field | Content | Purpose |
| :---- | :---- | :---- |
| **Vector** | \[0.12, \-0.05,...\] | **Search:** Allows finding relevant concepts. |
| **Metadata** | {"pub\_key": "..."} | **Identity:** Identifies who claims ownership. |
| **Payload** | { "data": "Encrypted..." } | **Provenance:** Opaque text. Unreadable without verifying. |

### **3.3 Phase 3: Retrieval & Verification (The "Read" Operation)**

When an AI Agent queries the RAG system:

1. **Query:** Agent sends query vector \-\> DB finds nearest neighbor vectors.  
2. **Return:** DB returns the Payload (Encrypted Chunk \+ Provenance Token).  
3. **Enforced Verification:**  
   * The Agent *cannot* read the text yet.  
   * Agent extracts public\_key and provenance\_token.  
   * **Operation:** RSA\_Public\_Decrypt(Provenance\_Token, Public\_Key).  
   * **Result:**  
     * *If Valid:* Returns $K\_{sym}$ (Session Key). Provenance is mathematically proven.  
     * *If Invalid:* Returns garbage. Access denied.  
4. **Decryption:** Agent uses $K\_{sym}$ to decrypt ciphertext \-\> obtains Plaintext Chunk.  
5. **Generation:** The plaintext is fed into the LLM Context Window.

## **4\. Implementation Specification (Python/Pseudo-code)**

This logic implements the "Redo" applied to a single 2KB chunk.

Python

from cryptography.hazmat.primitives import hashes, serialization  
from cryptography.hazmat.primitives.asymmetric import padding  
from cryptography.hazmat.primitives.ciphers.aead import AESGCM  
import os

class ProvenanceChunker:  
    def \_\_init\_\_(self, private\_key, public\_key):  
        self.private\_key \= private\_key  
        self.public\_key \= public\_key

    def process\_chunk(self, plaintext\_chunk: str):  
        \# 1\. Vectorization (Placeholder for embedding call)  
        \# vector \= embedding\_model.encode(plaintext\_chunk)  
          
        \# 2\. Hybrid Signcryption (The "Encrypt with Private Key" logic)  
          
        \# A. Generate ephemeral symmetric key (Session Key)  
        session\_key \= AESGCM.generate\_key(bit\_length=256)  
        aesgcm \= AESGCM(session\_key)  
        nonce \= os.urandom(12)  
          
        \# B. Encrypt the 2KB Data (Symmetric)  
        \# This keeps the data secret and manageable size-wise  
        ciphertext \= aesgcm.encrypt(nonce, plaintext\_chunk.encode('utf-8'), None)  
          
        \# C. "Encrypt" the Session Key with Private Key (Provenance Binding)  
        \# We use strict padding (PSS) to avoid security pitfalls  
        provenance\_token \= self.private\_key.sign(  
            session\_key, \# We sign the key itself  
            padding.PSS(  
                mgf=padding.MGF1(hashes.SHA256()),  
                salt\_length=padding.PSS.MAX\_LENGTH  
            ),  
            hashes.SHA256()  
        )  
        \# Note: To strictly follow "decrypt with public key" semantics,   
        \# one would use raw RSA primitives, but standard libraries force 'sign'.  
        \# The effect is identical: only PubKey can verify/unwrap.

        \# 3\. Construct the RAG Payload  
        return {  
            "vector": "vector\_data\_here",  
            "metadata": {  
                "owner\_id": "user\_123",  
                "pub\_key\_ref": "http://key-registry/user\_123.pem"  
            },  
            "provenance\_blob": {  
                "algo": "AES-256-GCM+RSA-2048",  
                "nonce": nonce.hex(),  
                "enc\_key": provenance\_token.hex(), \# The "Lock"  
                "data": ciphertext.hex()           \# The "Content"  
            }  
        }

## **5\. Scaling to 1MB Documents**

For a **1MB document**, the system will generate approximately **512 chunks** (assuming 2KB per chunk with no overlap, though 10-20% overlap is standard practice).

* **Batch Processing:** You do not need to generate a new RSA signature for every single chunk if they share the same provenance context.  
* **Optimization:** Generate *one* Session Key ($K\_{sym}$) for the entire 1MB document.  
  * "Encrypt" $K\_{sym}$ with the Private Key once \-\> Master\_Provenance\_Token.  
  * Encrypt all 512 chunks with the same $K\_{sym}$ (using unique nonces for each).  
  * Store the *same* Master\_Provenance\_Token in the metadata of all 512 vectors.  
  * *Benefit:* Reduces asymmetric operations from 512 to 1\.  
  * *Trade-off:* If the Master\_Provenance\_Token is cracked (unlikely), all 512 chunks are exposed. Given the security of RSA-2048, this is an acceptable risk for massive efficiency gains.

## **6\. Provenance Standards Alignment**

To ensure this custom architecture interacts with broader systems, the payload should be wrapped in a standard envelope.

### **6.1 C2PA (Content Credentials)**

While C2PA typically binds signatures to visible media, it can be adapted for text RAG:

* **Assertion:** Create a custom C2PA assertion labeled stds.schema-org.CreativeWork.  
* **Soft Binding:** The C2PA manifest acts as the "Provenance Token," containing the hash of the encrypted data.  
* **Verifiable RAG:** As noted in, "chunk-level provenance" is becoming a standard requirement for "Agentic RAG" to prevent hallucinations. Your architecture creates a stronger guarantee: **Cryptographic Access Control** at the chunk level.

## **7\. Conclusion**

To "redo" the provenance architecture for a 1MB document with 2KB RAG chunks:

1. **Abandon direct RSA encryption** of the chunks (2KB \> 245 bytes).  
2. **Adopt a Hybrid Wrapper:** Use the Private Key to lock a symmetric key, and use the symmetric key to lock the 2KB chunk.  
3. **Store Vectors \+ Blobs:** The Vector DB holds the map (vectors) and the locked chests (blobs).  
4. **Enforce Read-Time Verification:** The RAG retrieval process must include a client-side decryption step using the Public Key, ensuring that **accessing the data is mathematically equivalent to verifying its owner.**