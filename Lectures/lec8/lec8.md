# Topic 8. Digital Signatures. Message Authentication Codes.

## Lecture Plan


1. Signatures Algorithms.

2. RSA-based signatures. 

3. DSA and ElGamal.

4. Message Authentication Codes.

5. Qualified Electronic Trust Service Providers
---

## 1. Signatures Algorithms 


A **digital signature** is a cryptographic primitive that provides authenticity, integrity and non-repudiation for digital messages. Informally, a signature allows a signer holding a secret key to produce a short tag that any verifier holding the corresponding public key can check; no efficient adversary without the secret key should be able to produce a valid tag for a fresh message.

![1](https://upload.wikimedia.org/wikipedia/commons/7/78/Private_key_signing.svg)

#### Key Concepts

1. **Authenticity** – The signature confirms the sender's identity.
2. **Integrity** – Any modification of the message invalidates the signature.
3. **Non-repudiation** – The sender cannot deny having signed the message.


Digital signatures are based on **asymmetric (public-key) cryptography**, which uses a pair of mathematically related keys:

**Private key** – Used to generate the signature. Must remain secret.
**Public key** – Used to verify the signature. Can be shared openly.

#### General Process of Digital Signing

1. **Hashing the message**:  
   The sender computes a **hash** of the original message using a cryptographic hash function (e.g., SHA-256). The hash is a fixed-length representation of the message.

$$ H = \text{Hash}(M) $$

2. **Signing the hash**:  
   The sender encrypts the hash with their private key to generate the digital signature:

$$ S = \text{Encrypt}_{\text{PrivateKey}}(H) $$

3. **Sending the message**:  
   The sender transmits the original message $ M $ along with the signature $ S $.

4. **Verification**:  
   The recipient decrypts the signature using the sender’s public key to retrieve the hash:

$$ H' = \text{Decrypt}_{\text{PublicKey}}(S) $$

   Then, the recipient hashes the received message independently:

$$ H'' = \text{Hash}(M) $$

   If $ H' = H'' $, the signature is valid.

---

## 2. RSA-based signatures.

#### Textbook RSA Signatures

Let $N=pq$ and $(e,d)$ be RSA exponents with $ed\equiv 1\pmod{\varphi(N)}$.

- **Sign:** $\sigma \equiv M^d \pmod N$  
- **Verify:** $M \stackrel{?}{\equiv} \sigma^e \pmod N$

where $M=\mathsf{EMSA}(m)$ is the encoded message.

#### RSASSA-PSS / EMSA-PSS


**modBits** . The bit-length of the RSA modulus $n$ (i.e. $\lfloor \log_2 n\rfloor + 1$).

**emLen** (encoded-message length). The intended length in octets of the encoded message $EM$.

**OS2IP** *(Octet String to Integer Primitive)*. Interprets an octet string (byte sequence) as a nonnegative integer.

**I2OSP** *(Integer to Octet String Primitive)*. Converts a nonnegative integer into an octet string of a specified length.

**RSASP1**. The RSA signature primitive: given a private key $K = (n, d)$ and an integer message representative $m$, it computes

$$s = m^d \bmod n.$$

**RSAVP1**. The RSA verification primitive: given a public key $(n, e)$ and a signature representative $s$, it computes  

  $$m = s^e \bmod n.$$  

If $s \ge n$, then it is an error (“signature representative out of range”).

**Salt**. A random octet string of length `sLen` used in EMSA-PSS encoding to introduce randomness and probabilistic behavior.

**Trailer field** (for EMSA-PSS). A fixed octet, typically `0xBC`, appended at the end of the encoded message to mark structure. 

**EM** (encoded message). The result of EMSA-PSS-ENCODE, an octet string of length `emLen`. This is the value that is converted to an integer and signed.

**mHash**. The hash of the message:
  $$\mathrm{mHash} = \mathrm{Hash}(M).$$


RSA-PSS uses randomized padding with salt and masking:

- **Encode:**  
  1. Compute $\text{mHash} = \mathrm{Hash}(m)$.  
  2. Generate a random salt.  
  3. Compute $H = \mathrm{Hash}(\;0^8 || \text{mHash} || \text{salt}\;)$.  
  4. Construct DB = `PS || 0x01 || salt` (with zero padding `PS`).  
  5. Compute mask:  
     $$\text{dbMask} = \mathrm{MGF}(H,\; \text{emLen} - hLen - 1)$$  
     $$\text{maskedDB} = \text{DB} \oplus \text{dbMask}$$  
     $$\text{seedMask} = \mathrm{MGF}(\text{maskedDB},\; hLen)$$  
     $$\text{maskedSeed} = \text{salt} \oplus \text{seedMask}$$  
  6. Set leftmost unused bits of `maskedDB` to zero (if needed).  
  7. Form  
     $$\text{EM} = \text{maskedDB} || H || 0xBC.$$

- **Sign:**  
  $$\sigma = \mathrm{OS2IP}(EM)^d \bmod N$$  
  then convert to octets.

- **Verify:**  
  1. Compute $M' = \sigma^e \bmod N$ and convert to an octet string `EM'`.  
  2. Check `EM'` ends in `0xBC`, parse it into `maskedDB || H || 0xBC`.  
  3. Reverse masking, extract salt, verify padding `PS` and separator `0x01`.  
  4. Recompute hash `H` from message and salt, and check equality.

If all checks pass, signature is valid; otherwise reject.




#### RSASSA-PSS: Signature verification

**Procedure:** `RSASSA-PSS-VERIFY((n,e), M, S)`

**Input**
- $(n,e)$ — signer's RSA public key.  
- $M$ — message whose signature is to be verified (octet string).  
- $S$ — signature to verify, an octet string of length $k$, where $k$ is the length in octets of the RSA modulus $n$.

**Output**
- `"valid signature"` or `"invalid signature"`

**Steps**

1. **Length check.**  
   If the length of $S$ is not $k$ octets, output `"invalid signature"` and stop.

2. **RSA verification primitive.**  
   a. Convert $S$ to integer signature representative $s$:  
      $$s = \mathrm{OS2IP}(S).$$

   b. Apply the RSAVP1 verification primitive with public key $(n,e)$ to obtain integer message representative $m$:  
      $$m = \mathrm{RSAVP1}((n,e),\; s).$$  
      If RSAVP1 reports `"signature representative out of range"`, output `"invalid signature"` and stop.

   c. Convert $m$ to an encoded message $EM$ of length  
      $$\text{emLen} = \left\lceil(\text{modBits} - 1)/8\right\rceil$$  
      octets:  
      $$EM = \mathrm{I2OSP}(m,\; \text{emLen}).$$  
      *Note:* `emLen` is one less than $k$ if $\text{modBits}-1$ is divisible by $8$, otherwise it equals $k$. If `I2OSP` reports `"integer too large"`, output `"invalid signature"` and stop.

3. **EMSA-PSS verification.**  
   Compute:  
   $$\text{Result} = \text{EMSA-PSS-VERIFY}(M,\; EM,\; \text{modBits} - 1).$$

4. **Decision.**  
   If `Result = "consistent"`, output `"valid signature"`. Otherwise output `"invalid signature"`.


---

## 3. DSA and ElGamal


#### ElGamal Signature

The ElGamal signature scheme is a digital signature method introduced by Taher ElGamal in 1985, based on the hardness of the discrete logarithm problem. It should not be confused with ElGamal encryption (though they share the same author and underlying structure).

The scheme leverages the algebraic structure of modular exponentiation in a cyclic group and the difficulty of computing discrete logarithms in that group. In practice, ElGamal signatures are rarely used; instead, variants such as DSA (Digital Signature Algorithm) have become more widespread. Nonetheless, ElGamal signature remains a useful pedagogical example and an ancestor of many modern signature schemes.

##### Key Generation

1. **Choose parameters:**
Select a large prime number $ p $.
Choose a generator $ g $ of the multiplicative group $ \mathbb{Z}_p^* $.
Select a cryptographic hash function $ H $ with output length $ L $ bits.

2. **Generate keys:**
Choose a private key $ x \in \{1, \ldots, p-2\} $.
Compute the public key $ y = g^x \mod p $.


##### Signing Procedure

To sign a message $ m $:

1. **Select a random integer:**
Choose $ k \in \{1, \ldots, p-2\} $ such that $ \gcd(k, p-1) = 1 $.

2. **Compute signature components:**
$ r = g^k \mod p $.
If $ r = 0 $, choose a different $ k $.
Compute $ s = k^{-1} \cdot (H(m) - x \cdot r) \mod (p-1) $.
If $ s = 0 $, choose a different $ k $.

3. **Output the signature:**
The signature is the pair $ (r, s) $.


##### Verification Procedure

To verify a signature $ (r, s) $ for message $ m $:

1. **Check validity:**
Verify that $ 0 < r < p $ and $ 0 < s < p-1 $. If not, reject the signature.

2. **Compute verification components:** $ v_1 = g^{H(m)} \mod p $, $ v_2 = (y^r \cdot r^s) \mod p $.

3. **Compare and decide:** Accept the signature if $ v_1 \equiv v_2 \mod p $; otherwise, reject it.

#### DSA

DSA (Digital Signature Algorithm, 1991) is a modified and standardized version of ElGamal, approved by the NSA.

| Scheme  | Usage                          | Signature Size              | Notes              |
| ------- | ------------------------------ | --------------------------- | ------------------ |
| ElGamal | Rarely used                    | 2 numbers ≈ $\log_2 p$ bits | Historical scheme  |
| DSA     | FIPS 186 standard, widely used | 2 numbers ≈ $\log_2 q$ bits | Faster and smaller |

DSA is an optimized version of ElGamal for practical use.

Main differences:

- Smaller modulus $q$ instead of $p-1$.

- Slightly different signature formulas.

- DSA is standardized; ElGamal is mostly historical.


Let $p$ be prime, $q \mid (p - 1)$, and let $g$ be an element of order $q \bmod p$.

**Key generation**  
Choose a random private key  $x \in \mathbb{Z}_q$.
Compute the public key  $y = g^x \bmod p.$

**Signing** message $m$  
1. Pick a random nonce  $k \in \mathbb{Z}_q, \; k \neq 0.$
2. Compute  $r = (g^k \bmod p) \bmod q.$  If $r = 0$, retry with a new $k$.  
3. Compute  $s = k^{-1} \bigl(H(m) + x\,r \bigr) \bmod q.$ If $s = 0$, retry (choose new $k$).  
4. Signature is the pair $(r, s)$.

**Verification** of signature $(r, s)$ on message $m$  
1. Check that $r, s$ are in the valid range $[1, q-1]$. If not, reject.  
2. Compute  

$$w = s^{-1} \bmod q,$$  

$$u_1 = H(m)\,w \bmod q,\quad u_2 = r\,w \bmod q.$$  

3. Compute  

$$v = \bigl(g^{u_1} \cdot y^{u_2} \bmod p\bigr) \bmod q.$$  

4. Accept the signature if and only if  $v = r.$

“Textbook DSA” refers to the original DSA/ECDSA signature scheme where the per-signature secret nonce $k$ is chosen uniformly at random (from ${1,\dots,q-1}$) for each signature. If the randomness is bad (weak, biased, reused), then the private key $x$ can be leaked.

Non-textbook versions change the way $k$ is chosen, or add more structure around signing, so as to improve security, reduce dependence on randomness, provide determinism or better testability, etc.

[RFC 6979](https://www.ietf.org/rfc/rfc6979.txt) is a widely used specification that defines a deterministic process for generating $k$ in DSA/ECDSA instead of random.

Determinism means same signature for same message — this leaks some metadata: e.g. if someone sees duplicate signatures, they know the same message was signed. This may be undesirable in some privacy or voting protocols. 

| Feature                   | Textbook DSA/ECDSA                                  | RFC 6979 Deterministic Version                                                         |
| ------------------------- | --------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Nonce $k$                 | Random, fresh each signature                        | Derived deterministically from private key + message hash (possibly with other inputs) |
| Dependence on RNG         | Critical                                            | Only in key generation; signing does not need randomness                               |
| Signature uniqueness      | Different even on same message/key (if RNG differs) | Same signature for same message/key                                                    |
| Reproducibility / testing | Harder to test signatures (need good RNG)           | Easier: test vectors match exactly                                                     |
| Risk from bad RNG         | High (nonce reuse, leaks private key)               | Largely mitigated                                                                      |
| Privacy / anonymity       | Better (if signatures are random-looking)           | Slight leakage: duplicates of message => same signature                                |


## 4. Message Authentication Codes

A **Message Authentication Code (MAC)** is a cryptographic primitive that provides integrity and authenticity of a message. *Don't confuse with MAC address (medium access control address).* It is a short piece of information (a tag) generated from the message and a secret key. The MAC allows the receiver to verify that the message has not been altered and that it originates from a legitimate sender who possesses the secret key.

Message Authentication Codes are widely used in practice:
- To verify message integrity in communication protocols (TLS, IPSec).
- To authenticate data in storage systems.
- As building blocks in higher-level cryptographic primitives.

The differences between MACs and Digital Signatures:

| Feature              | MAC (Message Authentication Code)            | Digital Signature                       |
|----------------------|----------------------------------------------|------------------------------------------|
| **Key type**         | Symmetric – uses one shared secret key       | Asymmetric – uses private/public key pair |
| **Who can verify**   | Only someone who knows the secret key         | Anyone with the signer’s public key       |
| **Non-repudiation**  | ❌ No – both sides share the same key         | ✅ Yes – only signer has the private key   |
| **Provides**         | Integrity + authenticity                     | Integrity + authenticity + non-repudiation |
| **Speed & size**     | Faster, shorter tags                         | Slower, larger outputs                    |
| **Common algorithms**| HMAC (SHA-256, SHA-512, etc.)                | RSA, ECDSA, Ed25519                       |
| **Typical use**      | API auth, TLS message checks, internal systems| Contracts, software signing, certificates |


Formally, a MAC is a function of the form:

$$
\text{MAC} : \{0,1\}^* \times \{0,1\}^k \rightarrow \{0,1\}^n
$$

where:
- $\{0,1\}^*$ denotes the set of all finite binary strings (messages),
- $\{0,1\}^k$ is the space of secret keys,
- $\{0,1\}^n$ is the space of possible tags.

Let $M$ be the message space, $K$ the key space, and $T$ the tag space. A MAC consists of two efficient algorithms:

**Tag generation (signing)**:

$$
\text{Tag} \leftarrow \text{MAC}_{k}(m), \quad m \in M, \, k \in K
$$

**Verification**:

$$
\text{Verify}(k, m, t) =
\begin{cases}
\text{accept}, & \text{if } \text{MAC}_{k}(m) = t \\
\text{reject}, & \text{otherwise}
\end{cases}
$$

Thus, the receiver checks whether the received pair $(m,t)$ is valid with respect to the shared key $k$.

A secure MAC must satisfy the following properties:

1. **Unforgeability**

Without knowledge of the secret key $k$, it should be computationally infeasible to produce a valid tag $t$ for any new message $m$.  

Formally, given access to an oracle $\text{MAC}_k(\cdot)$, an adversary should not be able to generate $(m^*, t^*)$ such that:
   
$$
\text{Verify}(k, m^*, t^*) = \text{accept}
$$

   and $m^*$ was not previously queried to the oracle.

2. **Resistance to Key Recovery** 

The MAC function should not leak information about the secret key $k$.


#### Constructions of MAC
Several constructions of MACs are commonly used:

##### 1. MACs from Block Ciphers
Block ciphers (e.g., AES) can be used to build MACs. A prominent example is the **Cipher Block Chaining MAC (CBC-MAC)**.

For a message $m = m_1 \| m_2 \| \dots \| m_l$ divided into blocks of size $n$, the tag is computed as:

$$
C_0 = 0^n, \quad C_i = E_k(m_i \oplus C_{i-1}), \quad \text{for } i=1,\dots,l
$$
The final tag is:
$$
t = C_l
$$

##### 2. MACs from Hash Functions
A common construction is the **Hash-based MAC (HMAC)**, which uses a cryptographic hash function $H$.

The HMAC of a message $m$ under key $k$ is defined as:
$$
\text{HMAC}_k(m) = H\Big( (k' \oplus opad) \, \| \, H((k' \oplus ipad) \, \| \, m) \Big)
$$
where:
- $k'$ is a key derived from $k$,
- $opad$ and $ipad$ are fixed padding constants,
- $\|$ denotes concatenation.

##### 3. Universal Hash-Based MACs
Another class of MACs relies on universal hashing combined with a pseudorandom function. An example is **UMAC**.

---

## 5. Qualified Electronic Trust Service Providers

Digital Signature Providers, also known as Qualified Electronic Trust Service Providers (QETSPs), are organizations authorized to issue **Qualified Electronic Signatures (QES)**. These signatures are legally recognized and serve as a secure method for individuals and entities to authenticate their identity and ensure the integrity of electronic documents.

In Ukraine, the issuance of QES is regulated by the **Central Certification Authority (CCA)** under the Ministry of Digital Transformation. Providers must comply with national standards and are included in the official **Trusted List** published on the CCA's website.

[Examples of Qualified Electronic Trust Service Providers in Ukraine](https://czo.gov.ua/ca-registry):
PrivatBank, General Staff of the Armed Forces of Ukraine, General Prosecutor's Office of Ukraine, State Treasury of Ukraine, Diia.

| Algorithm                               | Key length / parameters | Where allowed / who uses it                                                                                                        | Advantages                                                                                                                                                                                                                                                    | Disadvantages                                                                                                                                                                                                                                                                                        |
| --------------------------------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **DSTU 4145-2002** (elliptic curves)    | 256 bit                 | QES (КНЕДП) “Diia” — one of the primary options for qualified certificates.                                                               | — Modern standard, good security properties at 256-bit ECC. <br>— Relatively compact keys and signatures compared with RSA. <br>— National (Ukrainian) standard, which makes regulatory compliance easier.                                                    | — Less widespread internationally than RSA or some other ECC schemes; may be less supported in some software solutions. <br>— Requires proper implementation (cryptographic primitives, key protection, etc.) to avoid vulnerabilities.                                                              |
| **ECDSA** (ECC, international standard) | 256 bit                 | Used as an alternative to DSTU 4145 in QES (КНЕДП) “Diia”; regulations were updated to support this algorithm.                            | — International recognition and compatibility (especially with the EU). <br>— Lower computational cost than RSA for a similar security level. <br>— Smaller signatures/certificates, more efficient transmission.                                             | — Like all ECC algorithms, sensitive to correct implementation (notably random-number generation and protection against side-channel attacks). <br>— Not all devices or software may provide support or hardware acceleration.                                                                       |
| **RSA-4096**                            | 4096 bit                | Allowed in QES (КНЕДП) “Diia” for qualified user and server certificates. Regulations also permit international algorithms RSA and ECDSA. | — Widely known and supported almost everywhere. <br>— High trust due to long history and extensive analysis. <br>— Less stringent requirements for randomness during signature generation than some ECC schemes (though proper randomness remains important). | — Very large keys, which lead to larger certificate and signature sizes and greater computational cost. <br>— Slower processing, especially on constrained devices. <br>— Potentially more vulnerable to future quantum attacks, although 4096-bit provides higher resistance than smaller RSA keys. |
