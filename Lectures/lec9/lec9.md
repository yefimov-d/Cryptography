# Topic 9. Elliptic Curve Cryptography

## Lecture Plan

1. Elliptic Curves

2. Group of Points on an Elliptic Curve

3. Standardized elliptic curves

4. Elliptic Curve Discrete Logarithm Problem (ECDLP)

5. Elliptic Curve Diffie–Hellman (ECDH) Key Exchange

6. Elliptic Curve Integrated Encryption Scheme (ECIES)

7. Elliptic Curve Digital Signature Algorithm (ECDSA)

---

## 1. Elliptic Curves

##### Elliptic curves over the real numbers

An **elliptic curve** is a set of points that satisfy a specific type of cubic equation.  
In the simplest and most common form (called the **Weierstrass form**), it is written as:

$$
y^2 = x^3 + ax + b
$$

where $a$ and $b$ are constants such that the curve has **no singularities** — that is, no cusps or self-intersections.  
This smoothness condition is ensured by the following **non-degeneracy requirement**:

$$
4a^3 + 27b^2 \ne 0
$$


Each point $(x, y)$ that satisfies the equation, together with a special point called the **point at infinity**, forms the set of points on the elliptic curve.

##### Example
For example, if we take

$$
y^2 = x^3 - x + 1,
$$

then all real points $(x, y)$ satisfying this equation, plus the point at infinity, make up the elliptic curve.

![real_curves](https://upload.wikimedia.org/wikipedia/commons/d/db/EllipticCurveCatalog.svg)
*Picture: Elliptic curves over the field of real numbers*


The term “elliptic” comes from the connection between these curves and **elliptic integrals**, not because the curves look like ellipses. Ellipses are not the elliptic curves.

*Above we have considered elliptic curves with the real coefficients.* In practice Elliptic Curve Cryptography uses **finite fields**. Two families are standard: curves over fields $\mathbb{F}_p$ and over fields $\mathbb{F}_{2^m}$.

##### Over field $\mathbb{F}_p$

**Fields** $\mathbb{F}_p$ with a large prime $p$ (p is odd). Most short Weierstrass curves used in cryptography are defined over $\mathbb{F}_p.$

The curve is written
$$
E: \; y^2 = x^3 + a x + b,\qquad a,b\in \mathbb{F}_p,
$$
with $4a^3+27b^2 \neq 0$.

![1](https://upload.wikimedia.org/wikipedia/commons/thumb/f/f4/Elliptic_curve_on_Z89.svg/1920px-Elliptic_curve_on_Z89.svg.png)
*Example: Set of affine points of elliptic curve $y^2 = x^3 − x$ over finite field $\mathbb{F}_{89}$.*

**There are other forms of elliptic curves**, such as Montgomery and Twisted Edwards curves. Note that over a field $ \mathbb{F}_p $ with $ p > 3 $, these forms are birationally equivalent to the Weierstrass form. Details: https://cr.yp.to/newelliptic/twisted-20080108.pdf

##### Over field $\mathbb{F}_{2^m}$

**Fields** $\mathbb{F}_{2^m}$. For these one usually uses the _binary_ curve equation (different from short Weierstrass). A common form is the _binary_ (or _non-supersingular_) model:

$$
E: \; y^2 + a_1 xy + a_3 y = x^3 + a_2 x^2 + a_4 x + a_6,\qquad a_i \in \mathbb{F}_{2^m},
$$

with [a suitable non-singularity condition.](https://www.johndcook.com/blog/2019/03/11/elliptic-curves-gf2-gf3/)

---

## 2. Group of Points on an Elliptic Curve

#### Over $\mathbb{F}_p$

For a field $K$ used in cryptography ($K=\mathbb{F}_p$ or $\mathbb{F}_{2^m}$) define
$$
E(K)=\{(x,y)\in K\times K \mid \text{(curve equation)}\}\cup\{\mathcal O\},
$$
where $\mathcal O$ is the point at infinity (the "group identity"/"zero").

We define an operation (+) on (E(K)) using the chord-and-tangent law.

Over $\mathbb{F}_p$ (odd characteristic) the chord-and-tangent law has the usual algebraic formulas.  
Let $P=(x_1,y_1),\; Q=(x_2,y_2)\in E(\mathbb{F}_p)$.

- Identity:

$$
P+\mathcal O = P.
$$

- Inverse:

$$
-(x,y) = (x,-y).
$$

- Distinct, non-vertical ($x_1\neq x_2$):

$$
\lambda = \frac{y_2-y_1}{x_2-x_1},\quad
x_3 = \lambda^2 - x_1 - x_2,\quad
y_3 = \lambda(x_1-x_3)-y_1,
$$

and $P+Q=(x_3,y_3)$ (all operations in $\mathbb{F}_p$).
- Doubling ($P=Q$, $y_1\neq 0$):

$$
\lambda = \frac{3x_1^2 + a}{2y_1},\quad
x_3 = \lambda^2 - 2x_1,\quad
y_3 = \lambda(x_1-x_3)-y_1.
$$

> Note: in $\mathbb{F}_p$ division means multiplying by the modular inverse. All formulas are reduced modulo $p$.

The operation ```+``` satisfies the abelian group axioms as follows:

| Axiom         | Property (Elliptic Curve Terms)              | Expression                                  |
| :------------ | :------------------------------------------- | :------------------------------------------ |
| Closure       | Adding two points gives another point on $E$ | $P, Q \in E(K) \implies P+Q \in E(K)$       |
| Identity      | The point at infinity acts as zero           | $P+\mathcal{O}=P$                           |
| Inverse       | Reflection over $x$-axis                     | $P+(-P)=\mathcal{O}$                        |
| Associativity | Grouping doesn’t matter                      | $(P+Q)+R = P+(Q+R)$                         |
| Commutativity | Order doesn’t matter                         | $P+Q = Q+P$                                 |

#### Special handling over $\mathbb{F}_{2^m}$ 

Over $\mathbb{F}_{2^m}$ the short Weierstrass form is not convenient; using the binary model

$$
y^2 + a_1 xy + a_3 y = x^3 + a_2 x^2 + a_4 x + a_6
$$

gives addition and doubling formulas that avoid dividing by `2` (which is zero in this case). The explicit formulas differ from the prime-field ones — implementers must use the correct binary formulas or projective variants designed for $\mathbb{F}_{2^m}$ (p. 331, ```An Introduction to Mathematical Cryptography, Jeffrey Hoffstein , Jill Pipher , Joseph H. Silverman```).

#### Numeric example

Take $p=17$ and curve

$$
E: y^2 = x^3 + 2x + 2 \pmod{17}.
$$

Check points $P=(5,1)$ and $Q=(6,3)$:
- $P$: $1^2 \equiv 5^3 + 2\cdot5 + 2 \equiv 1 \pmod{17}$, so $P\in E$.
- $Q$: $3^2 \equiv 6^3 + 2\cdot6 + 2 \equiv 9 \pmod{17}$, so $Q\in E$.

Compute $P+Q$ (distinct points):
- $\lambda = \dfrac{3-1}{6-5} \equiv \dfrac{2}{1} \equiv 2 \pmod{17}$,
- $x_3 \equiv 2^2 - 5 - 6 \equiv 4 - 11 \equiv -7 \equiv 10 \pmod{17}$,
- $y_3 \equiv 2\cdot(5-10) - 1 \equiv 2\cdot(-5)-1 \equiv -11 \equiv 6 \pmod{17}$.

So $P+Q=(10,6)$, and you can check $6^2\equiv 10^3+2\cdot10+2\pmod{17}$ holds.

Doubling $P$:
- $\lambda = \dfrac{3\cdot5^2 + 2}{2\cdot1} \equiv \dfrac{75+2}{2}\equiv \dfrac{77}{2}\pmod{17}$.
  Since $77\equiv 9$ and $2^{-1}\equiv 9$ (mod $17$), $\lambda\equiv 9\cdot9\equiv 13$.
- $x_3 \equiv 13^2 - 2\cdot5 \equiv 169 - 10 \equiv 6 \pmod{17}$,
- $y_3 \equiv 13\cdot(5-6) - 1 \equiv 13\cdot(-1)-1 \equiv -14 \equiv 3 \pmod{17}$.

So $2P=(6,3)$ (which equals $Q$ in this example), and algebra checks out.

---

## 3. Standardized elliptic curves

https://safecurves.cr.yp.to/

In practice, cryptography uses **well-studied, standardized elliptic curves** that are carefully chosen to avoid weaknesses and provide strong security.  
Practical curves are selected so that:
- the group $E(\mathbb{F}_m)$ has a large prime-order subgroup of size ≈ $2^{256}$ for 128-bit security,
- known attacks do not apply — this requires curve-specific checks.

In protocols one works in the subgroup generated by a base point $G$ of prime order $n$. Scalar multiplication $kG$ is the core operation; hardness of the elliptic-curve discrete logarithm problem (ECDLP) on that subgroup gives security.

Here we discuss some of the most famous curves, including Ed25519, secp256k1, and NIST P-curves.


#### edwards25519

- **Type:** Twisted Edwards curve  
- **Equation:** 
$$
-x^2 + y^2 = 1 + dx^2y^2, \quad d = -121665 \cdot 121666^{-1} \mod p
$$
with 
$$
p = 2^{255}-19
$$
- **Order:** 
$$
\#E = 8 \cdot l, \quad l \text{ prime } \approx 2^{252} + \dots
$$
- **Features:**  
  - Extremely fast and secure signature scheme (EdDSA).  
  - Designed to resist side-channel attacks.  
  - Small keys: 256-bit private keys, 256-bit public keys.  
  - Widely used in TLS, OpenSSH, Signal, and other applications.  
  - Ed25519 is a twisted Edwards curve birationally equivalent to Curve25519. 
- **Why famous:**  
  - High performance, high security, deterministic signatures, simple implementation.


#### Curve25519

- **Type:** Montgomery curve  
- **Equation:** 

$$
y^2 = x^3 + 486662 x^2 + x \mod p, \quad p = 2^{255}-19
$$

- **Use:**  
  - ECDH key exchange (Diffie–Hellman).  
  - Very fast and constant-time implementations.  
- **Relationship to Ed25519:**  
  - Same underlying field $p = 2^{255}-19$.  


#### secp256k1

- **Type:** Short Weierstrass curve  
- **Equation:** 
$$
y^2 = x^3 + 7 \mod p, \quad p = 2^{256} - 2^{32} - 2^9 - 2^8 - 2^7 - 2^6 - 2^4 - 1
$$
- **Order:** 
$$
\#E = n = 2^{256} - 432420386565659656852420866394968145599
$$

Read more [here](https://en.bitcoin.it/wiki/Secp256k1#:~:text=Properties%20*%20secp256k1%20has%20characteristic%20p%2C%20it,equation%20becomes%20y2%20=%20x3%20+%207.).

- **Features:**  
  - Used by Bitcoin and other cryptocurrencies.  
  - Chosen for efficient computation of scalar multiplication.  
  - No known weaknesses; not standardized by NIST, but widely trusted in crypto communities.  
- **Why famous:**  
  - Core of Bitcoin’s security and blockchain signatures (ECDSA).


#### NIST P-Curves (P-256, P-384, P-521)

https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-186.pdf

- **Type:** Short Weierstrass curves  

- **Features:**  
  - Officially standardized by NIST and widely supported in TLS/SSL and cryptographic libraries.  
  - Provide 128-bit (P-256), 192-bit (P-384), and 256-bit (P-521) security levels.  
  - Trusted, but implementation errors and some suspicion about parameter generation have led some developers to prefer alternative curves (like Ed25519).  


**Summary**


**Ed25519**: signature-focused, very fast, secure, resistant to side-channel attacks.  
**Curve25519**: key exchange-focused, highly efficient, constant-time scalar multiplication.  
**secp256k1**: used in cryptocurrencies; fast Weierstrass curve, widely deployed.  
**NIST P-curves**: standardized, widely supported, varying security levels.  

All these curves are designed so that solving the **ECDLP** is infeasible, providing the foundation for modern elliptic curve cryptography.


---
## 4. Elliptic Curve Discrete Logarithm Problem (ECDLP)

The **Elliptic Curve Discrete Logarithm Problem (ECDLP)** is the mathematical foundation of elliptic curve cryptography (ECC). This problem is the cornerstone of all modern elliptic curve cryptosystems.

Let $E$ be an elliptic curve defined over a finite field $\mathbb{F}_q$. Let $P \in E(\mathbb{F}_q)$ be a point of **prime order** $n$.  
This means that repeated addition of $P$ eventually gives the point at infinity $\mathcal{O}$:

$$
nP = \mathcal{O}.
$$

For any integer $k$ with $0 \le k < n$, define

$$
Q = kP.
$$

The **Elliptic Curve Discrete Logarithm Problem (ECDLP)** is:

> Given $P$ and $Q$, find the integer $k$ such that $Q = kP$.

#### Why It Is Hard

The operation of computing $Q = kP$ (called **scalar multiplication**) is easy and efficient,  
since it can be done using the **double-and-add algorithm** in $O(\log k)$ group operations.

However, the **inverse operation** — finding $k$ given $P$ and $Q$ — has no known efficient algorithm.  
All known methods require **exponential time** in the size of $n$.

This asymmetry (easy in one direction, hard in the other) is what provides security.

The classical **Discrete Logarithm Problem (DLP)** in a finite field $\mathbb{F}_p^\times$ is:

$$
g^k \equiv h \pmod{p}.
$$

Here we know $g$, $h$, and want to find $k$. The ECDLP is similar, but group operations are point additions on a curve instead of modular multiplications.

| Algorithm | Complexity | Memory | Notes |
|------------|-------------|---------|-------|
| Brute-force search | $O(n)$ | small | Completely impractical |
| Baby-Step Giant-Step (BSGS) | $O(\sqrt{n})$ | $O(\sqrt{n})$ | Deterministic |
| Pollard’s Lambda | $O(\sqrt{n})$ | small | Useful when $k$ in known range |

The ECDLP is believed to require roughly $\sqrt{n}$ operations to solve.

#### Security Implications

For a curve with a subgroup of prime order $n \approx 2^{256}$, the best known attack (Pollard’s rho) would require about $2^{128}$ operations. This level of difficulty provides **128-bit security**, which is comparable to RSA with a modulus of about $3072$ bits. Thus, elliptic curve cryptography achieves **strong security with shorter keys**.

*Source:* [NIST SP 800-57 Part 1 Rev. 5](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-57pt1r5.pdf)


| Security Strength (bits) | Symmetric Key Algorithms | FFC (DSA, DH, MQV) | IFC* (RSA) | ECC* (ECDSA, EdDSA, DH, MQV) |
| ------------------------ | ------------------------ | ------------------ | ---------- | ---------------------------- |
| 80                       | Skipjack, 2TDEA          | L = 1024, N = 160  | k = 1024   | f = 160–223                  |
| 112                      | 3TDEA                    | L = 2048, N = 224  | k = 2048   | f = 224–255                  |
| 128                      | AES-128                  | L = 3072, N = 256  | k = 3072   | f = 256–383                  |
| 192                      | AES-192                  | L = 7680, N = 384  | k = 7680   | f = 384–511                  |
| 256                      | AES-256                  | L = 15360, N = 512 | k = 15360  | f = 512+                     |

*Explanation*

- The parameters L, N, k, and f denote bit lengths of modulus or field size.

- Security Strength: Number of bits of symmetric key providing equivalent strength.

- FFC: Finite Field Cryptography (DSA, Diffie–Hellman, MQV).

- IFC: Integer Factorization Cryptography (RSA).

- ECC: Elliptic Curve Cryptography (ECDSA, EdDSA, DH, MQV).


The **Elliptic Curve Discrete Logarithm Problem (ECDLP)** itself is not something you “solve” in practice (because it is supposed to be hard).  However, there are **software packages and libraries** that implement elliptic curve operations, scalar multiplication, and cryptographic primitives based on ECDLP (like ECDSA, ECDH, ECIES). Some tools are also used for **research and testing attacks** on small curves to study ECDLP hardness.

##### Python Packages

| Package | Description | Notes |
|---------|------------|------|
| `ecdsa` | Pure Python ECDSA and ECDH implementation | Can compute scalar multiplication, signatures, key generation |
| `tinyec` | Simple elliptic curve library for Python | Educational; supports point addition, doubling, scalar multiplication |
| `pycryptodome` | Cryptography library (fork of PyCrypto) | Implements ECC for ECDSA, ECDH, etc. |
| `cryptography` | Python bindings for OpenSSL | Supports standardized curves like NIST P-256, P-384, X25519 |
| `sage` (SageMath) | Advanced algebra and number theory | Can implement small-scale ECDLP experiments, count points, test attacks |

##### C/C++ Packages

| Library           | Description                                                   |
| ----------------- | ------------------------------------------------------------- |
| ```OpenSSL```           | Widely used; supports ECDSA, ECDH, ECIES, standardized curves |
| ```libsodium```         | High-level crypto library; supports X25519 and Ed25519        |
| ```libsecp256k1```      | Optimized for Bitcoin; implements secp256k1 curve efficiently |
| ```GMP + custom code``` | Arbitrary precision arithmetic for small curve experiments    |

Hardware wallets (Bitcoin, Ethereum) use secp256k1 and ```Ed25519``` curves — rely on ECDLP hardness for private key protection.

GPU/FPGA implementations are used by researchers to test attacks on small curves or for cryptanalysis experiments.

---
## 5. Elliptic Curve Diffie–Hellman (ECDH) Key Exchange


ECDH is a way for **two parties** (Alice and Bob) to agree on a shared secret over an insecure channel using elliptic curves.


#### Protocol Flow

1. Alice → Bob: send $A = aG$.  
2. Bob → Alice: send $B = bG$.  
3. Alice computes $S = aB$.  
4. Bob computes $S = bA$.  
5. Derive a symmetric key from $S$:
$$
K = \mathrm{KDF}(\mathrm{Encode}(S)\,\|\,\text{context})
$$



#### Toy Example (Educational Only)

Curve: $y^2 = x^3 + 2x + 2 \mod 17$, base $G=(5,1)$.

- Alice: $a=3$, $A=3G=(10,6)$  
- Bob: $b=2$, $B=2G=(6,3)$  

Compute shared secret:

- Alice: $S = aB = 3*(6,3) = (16,13)$  
- Bob: $S = bA = 2*(10,6) = (16,13)$  

Both share $S=(16,13)$. In a toy KDF, take $x(S)=16$ as key.


#### Practical Choices

**Curves:** X25519 (simple, safe), P-256 (NIST)  
**KDF:** HKDF-SHA256  
**Usage:** combine ECDH with AEAD (AES-GCM, ChaCha20-Poly1305)  
**Libraries:** `cryptography`, `libsodium`, `OpenSSL` 



#### Security Considerations

- Validate received points (on-curve, correct order).  
- Handle cofactors correctly.  
- Use ephemeral keys for forward secrecy.  
- Use constant-time implementations to avoid side-channels.  
- Derive keys with KDF and include context info.  


#### Simple Pseudocode

```
# Alice
a = random_scalar()
A = a * G
send A
receive B
S = a * B
K = HKDF(Encode(S), info)
# use K for secure communication

# Bob
b = random_scalar()
B = b * G
send B
receive A
S = b * A
K = HKDF(Encode(S), info)
```

---

## 6. Elliptic Curve Integrated Encryption Scheme (ECIES)

#### Short explanation

- ECIES (Elliptic Curve Integrated Encryption Scheme) is a way to **encrypt a message using someone’s elliptic-curve public key**.  
- It is a *hybrid* scheme: an elliptic-curve operation (ECDH) makes a shared secret, a KDF turns that secret into symmetric keys, and a fast symmetric cipher does the real encryption.  
- The sender creates a fresh **ephemeral** key for every message. This gives **forward secrecy**: even if the sender’s ephemeral key leaks later, it does not compromise other messages.

Rough flow (one sentence): sender picks ephemeral key $d_{eph}$, computes $R=d_{eph}G$, computes shared secret $S=d_{eph}Q_{rec}$, derives symmetric key(s) $K=\mathrm{KDF}(S)$, encrypts message with $K$, sends $(R,\text{ciphertext})$.


#### Why the parts are needed (intuitively)

- **ECDH (KEM)**: fast one-time key agreement — both parties compute same secret $S$ but an eavesdropper cannot (ECDLP hardness).  
- **KDF**: turns the elliptic-curve point $S$ into uniformly random-looking bytes.  
- **DEM (symmetric cipher)**: encrypts the bulk data efficiently (AES-GCM or ChaCha20-Poly1305).  
- **MAC / AEAD**: prevents tampering.



#### Pseudocode (conceptual)

```text
# Sender (encrypt)
d_eph = random_scalar()
R = d_eph * G
S = d_eph * Q_rec           # ECDH
s_bytes = Encode(S)         # e.g. x-coordinate or full point encoded
K = HKDF(s_bytes, info)     # derive keys
C = AEAD_Encrypt(K, M)      # encrypt (and authenticate)
send(Encode(R) || C)

# Receiver (decrypt)
R = Decode(received_R)
S' = d_rec * R
s_bytes' = Encode(S')
K' = HKDF(s_bytes', info)
M = AEAD_Decrypt(K', C)     # verify tag + decrypt
```

---

## 7. Elliptic Curve Digital Signature Algorithm (ECDSA)

https://datatracker.ietf.org/doc/html/rfc8032

ECDSA is a widely used algorithm for creating **digital signatures** with elliptic curves. It allows a user to **prove ownership of a private key** and verify messages without revealing the private key.



#### Key Components

- **Curve:** $E(\mathbb{F}_q)$, base point $G$ of order $n$.
- **Private key:** $d \in [1, n-1]$.
- **Public key:** $Q = dG$.
- **Hash function:** $H(\cdot)$ (SHA-256 or similar) to convert messages into fixed-size numbers.


#### Signature Generation

To sign a message $m$:

1. Compute hash:
$$
h = H(m)
$$
2. Choose a random ephemeral scalar $k \in [1, n-1]$.  
3. Compute ephemeral point:
$$
R = kG = (x_R, y_R)
$$
4. Compute
$$
r = x_R \bmod n
$$
   - If $r = 0$, pick another $k$.  
5. Compute
$$
s = k^{-1} (h + d \cdot r) \bmod n
$$
   - If $s = 0$, pick another $k$.  

**Signature:** $(r, s)$

> Note: $k$ must be random and secret for each signature. Reusing $k$ leaks the private key.


#### Signature Verification

Given message $m$, signature $(r, s)$, and public key $Q$:

1. Verify $r, s \in [1, n-1]$.  
2. Compute hash:
$$
h = H(m)
$$
3. Compute
$$
w = s^{-1} \bmod n
$$
4. Compute
$$
u_1 = h \cdot w \bmod n, \quad u_2 = r \cdot w \bmod n
$$
5. Compute point
$$
X = u_1 G + u_2 Q = (x_X, y_X)
$$
6. Accept signature if
$$
r \equiv x_X \bmod n
$$


#### Intuition

- $kG$ creates a unique “ephemeral” point for this signature.  
- Only the holder of private key $d$ can compute $s$ that matches $r$.  
- Verifier uses $Q$ to check that the signature is consistent with the message.



| Step | Purpose |
|------|---------|
| KeyGen | generate private $d$ and public $Q = dG$ |
| Sign | compute $(r,s)$ using $k$ and $d$ |
| Verify | check $(r,s)$ using $Q$ and $m$ |
| Security | relies on ECDLP and unique random $k$ |



#### Analogy

- Imagine each message gets a **lockbox** created from ephemeral $k$.  
- Only someone with the private key $d$ can **compute the right combination** to lock/unlock the box.  
- Verifier uses the public key to check that the combination is correct **without knowing the private key**.


#### Summary

- ECDSA = elliptic-curve digital signature algorithm.  
- Provides authenticity and integrity of messages.  
- Key points: private key $d$, public key $Q$, ephemeral $k$, hash of message $h$, signature $(r,s)$.  
- Always use modern curves, secure random numbers, and well-tested libraries.

