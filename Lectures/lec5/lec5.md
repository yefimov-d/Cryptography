# Topic 5. **Hash Functions**

## Lecture Plan

1. Cryptographic hash functions.

2. Message padding methods

3. Merkle–Damgård construction and Sponge function

4. Hash functions MD4, MD5, SHA-1, SHA-2, SHA-3, Kupyna

5. Applications: Password storage, Proof of Work, Merkle tree.

---

# 1. Cryptographic hash functions. Birthday paradox

Cryptographic hash functions are deterministic algorithms that take an input of arbitrary length and produce a fixed-length output called a *hash* or *digest*. Formally, a hash function is often written as

$$
h : \{0,1\}^* \to \{0,1\}^n,
$$

where $n$ is the output length in bits. Good cryptographic hash functions are designed so that their outputs look random and small changes in the input produce large, unpredictable changes in the digest.

#### Key security properties
A cryptographic hash function $h$ is useful in practice when it approximately satisfies three properties:

- **Preimage resistance (one-wayness):** Given a digest $y$, it should be computationally infeasible to find any $x$ with $h(x)=y$.
- **Second-preimage resistance:** Given an input $x$, it should be infeasible to find $x'\ne x$ such that $h(x')=h(x)$.
- **Collision resistance:** It should be infeasible to find any distinct pair $(x,x')$ with $h(x)=h(x')$.

The strength of collision resistance is usually measured in terms of bit-security: for an $n$-bit hash the generic attack cost for finding a collision by the birthday paradox is about $2^{n/2}$ operations.

#### Typical uses in systems
**Data integrity.** Verify that a message or file has not changed.

**Password storage.** Store salted and stretched hashes rather than plaintext passwords.

**Message authentication (with a key).** HMAC and related constructions.

**Merkle trees.** Compactly commit to large sets of data (used in logging, blockchains, P2P systems).

**Proof-of-work.** Hash inversion as a computational puzzle (e.g., Bitcoin uses repeated SHA-256).

**Digital signatures and commitments.** Hash-then-sign pattern, hash-based signatures (Lamport, Winternitz).

#### Collisions and Birthday paradox

The *birthday paradox* is the counter-intuitive fact that in a set of randomly chosen items from a finite space, a collision (two items that match) becomes likely much sooner than the size of the space. For example, with 23 people the chance two share a birthday exceeds 50% even though there are 365 possible birthdays. In the context of hash functions this shows why collisions in an $n$-bit hash do **not** require $2^n$ trials but only on the order of $2^{n/2}$ trials (the so-called birthday bound).

**Generic cost to find any collision** for an $n$-bit hash is about $2^{\,n/2}$ hash evaluations (this is the classical, generic/birthday attack cost — i.e. the best one can do without exploiting structure-specific weaknesses).

**Interpretation:** an $n$-bit output gives roughly $n/2$ bits of *collision security*.  
  - A 256-bit hash gives about $128$-bit collision security (≈ $2^{128}$ work).  
  - A 128-bit hash gives about $64$-bit collision security (≈ $2^{64}$ work).

For $k$ randomly chosen digests from a space of size $2^n$ the collision probability is approximately

$$
P(\text{collision}) \approx 1 - \exp\!\left(-\frac{k(k-1)}{2^{\,n+1}}\right).
$$

If you choose $k=2^{\,n/2}$ digests, the collision probability is

$$
P \approx 1 - e^{-1/2} \approx 0.6321 \quad(\text{about }63.21 \%).
$$


The number of digests $k$ required for a **50%** chance of at least one collision is

$$
k_{50\%} \approx \sqrt{2\ln 2}\;\; 2^{\,n/2} \approx 1.17741\;2^{\,n/2}.
$$

So you need only about $1.177\times 2^{n/2}$ items for a coin-flip chance of collision.

---

# 2. Message padding methods

Suppose we begin with a message $m$ of length $\ell$ bits. We want to adjust it so that its total length becomes exactly $k \cdot b$ bits. This is achieved through *padding*, where extra bits are appended until the required size is reached. There are different ways to perform this operation; here we describe five methods. The padded message is written as  

$$
m \parallel \text{pad}_i(|m|, b).
$$  

The padding methods are defined as follows:  

- **Method 0:** Let $v = b - |m| \pmod{b}$. Append $v$ zero bits to the message:  

$$
m \parallel \text{pad}_0(|m|, b) = m \parallel 0^*.
$$  

- **Method 1:** Let $v = b - (|m| + 1) \pmod{b}$. Add a single $1$ bit, then $v$ zeros: 

$$
m \parallel \text{pad}_1(|m|, b) = m \parallel 10^*.
$$  

- **Method 2:** Let $v = b - (|m| + 65) \pmod{b}$. Represent $|m|$ as a 64-bit integer $\ell$. Append one $1$ bit, then $v$ zeros, followed by $\ell$:  

$$
m \parallel \text{pad}_2(|m|, b) = m \parallel 10^* \parallel \ell.
$$  

- **Method 3:** Let $v = b - (|m| + 64) \pmod{b}$. Encode $|m|$ as a 64-bit integer $\ell$. Append $v$ zeros, then $\ell$: 

$$
m \parallel \text{pad}_3(|m|, b) = m \parallel 0^* \parallel \ell.
$$  

- **Method 4:** Let $v = b - (|m| + 2) \pmod{b}$. Add one $1$ bit, then $v$ zeros, and finally another $1$: 

$$
m \parallel \text{pad}_4(|m|, b) = m \parallel 10^*1.
$$  

---

# 3. Merkle–Damgård construction and Sponge function

### Merkle–Damgård construction

The **Merkle–Damgård construction** is a method for building cryptographic hash functions from a fixed-length compression function. It has become the foundation for many well-known hash functions, including MD5, SHA-1, and the SHA-2 family.


![1](https://upload.wikimedia.org/wikipedia/commons/e/ed/Merkle-Damgard_hash_big.svg)



$l = s - n$

Pad the input message $m$ with zeros so that it is a multiple of $l$ bits in length.  

Divide the input $m$ into $t$ blocks of $l$ bits long, $m_1, \ldots, m_t$.  

Set $H$ to be some fixed bit string of length $n$.  

**for** $i = 1$ **to** $t$ **do**  
&emsp; $H = f(H \parallel m_i)$  
**end**  

**return** $(H)$


If the compression function $f$ is collision-resistant, then the hash function built with the Merkle–Damgård construction is also collision-resistant.
 
Hash functions based on this design are subject to the *length-extension attack*. Given $H(m)$ and the length of $m$, one can compute $H(m \parallel m')$ for some extra block $m'$ without knowing $m$. This issue affects MD5, SHA-1, and SHA-2.

Despite its weaknesses, the Merkle–Damgård paradigm underlies most classical hash functions before SHA-3, which instead uses the sponge construction.

The Merkle–Damgård construction provides a systematic way to transform a fixed-size compression function into a variable-length hash function. Its simplicity and provable security properties made it the standard approach for decades, but modern designs are moving toward alternatives such as sponge functions to mitigate its limitations.


### Sponge function

The sponge is a simple, powerful template: absorb input into a $b$-bit state via XOR+permutation, then squeeze output. By separating throughput ($r$) and security ($c$) it provides flexible, robust building blocks for modern hash/XOF/MAC/KDF constructions — SHA-3/Keccak is the canonical practical instance.


![1](https://upload.wikimedia.org/wikipedia/commons/7/70/SpongeConstruction.svg)

Generic keyed or unkeyed primitive that absorbs input into a large internal state and then squeezes output from it.  

Core parameters: **rate** $r$ (bits processed per permutation call) and **capacity** $c$ (security-related bits), with state size $b = r + c$. 

Core primitive: a fixed-width **permutation** or **f**-function $P$ acting on the $b$-bit state.  

Used by: SHA-3 / Keccak (sponge with Keccak-$f$), SHAKE XOFs, and many designs (MACs, KDFs, PRGs) built on sponge.


---

# 4. Hash functions MD5, SHA-1, SHA-2, Keccak and SHA-3, Kupyna

The **MD-family** (Message Digest family) includes hash functions such as **MD4, MD5, SHA-1 and SHA-2**. They share a common structure called the **Merkle–Damgård construction** (see above), which processes input in fixed-size blocks through multiple **rounds** of **steps** using a **compression function**.    
SHA-3 (based on the **Keccak sponge construction**) differs fundamentally from MD-family hashes. Instead of the Merkle–Damgård structure, it uses a **sponge function**:  

1. **Absorbing phase**. Input blocks are XORed into a fixed-size internal state, which is repeatedly permuted.  
2. **Squeezing phase**. The internal state is used to produce the hash output, potentially of arbitrary length.  

Key properties of the Keccak sponge:  

1. No fixed “rounds” like MD-family hashes; instead, the state undergoes repeated permutations.  
2. Flexible output size (SHA-3 supports 224, 256, 384, 512 bits).  
3. Very strong resistance to collision and preimage attacks.  

#### MD4

**Quick facts**
Designer: Ron Rivest (1990).  
Output size: $128$ bits.  
Input block size: $512$ bits (processed as sixteen $32$-bit little-endian words).  
Construction: Merkle–Damgård (iterated compression function).  
Status: **cryptographically broken** — unsuitable for new systems.

**High-level idea**
MD4 compresses each $512$-bit block to update a $128$-bit internal state $(A,B,C,D)$. After padding the message and processing all blocks, the final $A\|B\|C\|D$ (little-endian) is the $128$-bit digest.

Details: https://datatracker.ietf.org/doc/html/rfc1186

##### Important properties

**Fast** on 32-bit little-endian CPUs (few simple operations per step).  

**Merkle–Damgård** implies MD4 is vulnerable to length-extension attacks: given $\text{MD4}(m)$ and $|m|$ one can compute $\text{MD4}(m\|suffix)$ without knowing $m$.  

**Broken collision resistance:** practical collision attacks exist; MD4 should **not** be used where collision resistance or strong preimage resistance matters.

**Legacy use:** appears in some legacy systems (e.g., certain password/hash schemes), which is why it still appears in discussions — but it is deprecated for new designs.


MD4 is an important historical design: simple, influential (inspired MD5 and others) and helpful to study because its small size makes the structure and attacks clear. However, from a security standpoint it is obsolete — prefer SHA-2 / SHA-3 families for real applications.


#### MD5

**Quick facts**
Designer: Ron Rivest (1990).  
Output size: $128$ bits.  
Input block size: $512$ bits (processed as sixteen $32$-bit little-endian words).  
Construction: Merkle–Damgård (iterated compression function).  
Status: **cryptographically broken** — unsuitable for new systems.

**High-level idea**
MD5 updates a $128$-bit internal state $(A,B,C,D)$ by processing the padded message block-by-block. Each $512$-bit block is parsed into words $X_0,\dots,X_{15}$ and fed into a compression that runs $64$ steps grouped into 4 rounds. Every step uses a nonlinear boolean function, a per-step constant $T_i$, one message word $X_k$, addition modulo $2^{32}$ and a left rotation. After all blocks are processed the concatenation $A\|B\|C\|D$ (little-endian) is the $128$-bit digest.


More details: https://www.rfc-editor.org/rfc/rfc1321

Practical collision attacks exist: attackers can find distinct messages $m\neq m'$ with $\mathrm{MD5}(m)=\mathrm{MD5}(m')$ far faster than the birthday bound. For a $128$-bit hash the birthday bound is $\approx 2^{128/2}=2^{64}$, but practical attacks require far fewer resources than $2^{64}$.

Recommendation: do **not** use MD5 for cryptographic purposes that require collision resistance, digital signatures, or secure file integrity. Use SHA-256 / SHA-3 (or later families) for general hashing, and use dedicated password/hash functions (bcrypt/scrypt/argon2) for passwords and HMAC with a secure hash for MACs.


#### SHA-1

**Quick facts**
Designer / Standard: Developed by the NSA; specified in FIPS PUB 180 (1995) and updated later. 

Output size: $160$ bits.  

Input block size: $512$ bits (processed as sixteen $32$-bit big-endian words). 

Construction: Merkle–Damgård (iterated compression function).  

Internal structure: $80$ steps (rounds) organized conceptually into 4 groups of $20$ steps, with a word expansion $W_0,\dots,W_{79}$.  

Status: **cryptographically broken for collision resistance** — deprecated for new systems.

**High-level idea**
SHA-1 maintains a $160$-bit internal state $(H_0,H_1,H_2,H_3,H_4)$. The message is padded and split into $512$-bit blocks; each block is parsed into $16$ big-endian $32$-bit words, expanded to $80$ words, and processed by an $80$-step compression function that mixes the state with the message words using simple bitwise functions, modular addition, and rotations. After all blocks are processed the final digest is the concatenation $H_0\|H_1\|H_2\|H_3\|H_4$ (each word in big-endian), giving a $160$-bit hash.

More details: https://datatracker.ietf.org/doc/html/rfc3174


Cryptanalytic advances starting in the mid-2000s showed practical weaknesses in SHA-1's collision resistance. Subsequent work produced concrete collisions (demonstrations such as the SHAttered collision) and further optimizations reduced the cost of collisions below the theoretical birthday bound.

Like other Merkle–Damgård hashes, given $h=\mathrm{SHA1}(m)$ and $|m|$ an attacker can compute $\mathrm{SHA1}(m\parallel\mathrm{pad}(m)\parallel \text{suffix})$ without knowing $m$. This breaks naive MAC constructions $\mathrm{SHA1}(k\parallel m)$ — use HMAC-SHA1 or, better, HMAC with SHA-2/3.

Do **not** use SHA-1 for new cryptographic applications that require collision resistance (digital signatures, certificate fingerprints, file integrity for adversarial settings). 

#### SHA-2

**Quick facts**
Designer / Standard: Designed by the NSA; standardized as part of the SHA-2 family in FIPS PUB 180-2 (2001) and later updates. 

Common variants and output sizes: $\text{SHA-224}$ (224 bits), $\text{SHA-256}$ (256 bits), $\text{SHA-384}$ (384 bits), $\text{SHA-512}$ (512 bits). There are also truncated variants $\text{SHA-512/224}$ and $\text{SHA-512/256}$.  

Block / word sizes:  
  - SHA-224 / SHA-256: block size $512$ bits, word size $32$ bits, $64$ rounds.  
  - SHA-384 / SHA-512: block size $1024$ bits, word size $64$ bits, $80$ rounds.  

Construction: Iterated compression (Merkle–Damgård-style) with a message schedule and round constants derived from fractional parts of prime roots.  

Status: **Secure and widely used** (SHA-2 remains a standard recommendation for most applications; SHA-256 and SHA-512 are common).

**High-level idea**
SHA-2 maintains a state of eight words $(a,b,c,d,e,f,g,h)$ (each $32$ or $64$ bits depending on variant). After standard padding and parsing, each message block is expanded into a message schedule $W_t$ and processed by a fixed number of rounds. Each round mixes the state with one schedule word and a round constant using a few simple bitwise functions, rotations, and modular additions. A typical round (conceptual) computes two temporary values and updates the state:

SHA-2 is the workhorse family of modern hashing: clean design, clear security margins tied to output size, fast implementations, and broad standard support. Use SHA-256 or SHA-512 (plus HMAC for MACs) in real systems — avoid older hashes like MD5 or SHA-1 for security-sensitive tasks.


#### SHA-3

**Quick facts**

Standard / Year: Keccak was selected as SHA-3 in the NIST competition; standardized in FIPS 202 (2015).  

Family / outputs: $\mathrm{SHA3\text{-}224},\;\mathrm{SHA3\text{-}256},\;\mathrm{SHA3\text{-}384},\;\mathrm{SHA3\text{-}512}$. Also extendable-output functions (XOFs) $\mathrm{SHAKE128}$ and $\mathrm{SHAKE256}$ which produce variable-length output.  


More details: https://csrc.nist.gov/csrc/media/Projects/hash-functions/documents/Keccak-submission-3.pdf 

Status: **Modern and secure (as of standardization)** — different internal design from SHA-2, widely recommended where its properties are useful.

Example of lavine effect:

```
SHA3-256("The quick brown fox jumps over the lazy dog")
69070dda01975c8c120c3aada1b282394e7f032fa9cf32f4cb2259a0897dfc04
SHA3-256("The quick brown fox jumps over the lazy dog.")
a80f839cd4f83f6c3dafc87feae470045e4eb0d366397d5c6ce34ba1739f734d
```

#### Kalyna

**Quick facts**
Standard / Year: adopted as the Ukrainian national block-cipher standard **DSTU 7624:2014** (published 2015). 

Designer / origin: result of the Ukrainian national public cryptographic competition (designers from Ukrainian crypto community).   
Block sizes: $128,\;256,\;\text{or}\;512$ bits.  

Key sizes: $128,\;256,\;\text{or}\;512$ bits (key length equals block length or is double the block length in some parameter choices). 

Structure: Substitution–Permutation Network (SPN), AES-like round structure but with different S-boxes, larger MDS linear layer and a distinct key schedule. 

Number of rounds: depends on parameters — typically $10$ rounds (for 128-bit keys), $14$ rounds (for 256-bit keys), $18$ rounds (for 512-bit keys). 


Kalyna is an AES-style SPN chosen as Ukraine's national cipher standard. It supports large block and key sizes ($128$–$512$ bits) and uses distinct S-boxes, MDS linear layers and a specialized key schedule to achieve diffusion and security for those sizes. While reduced-round cryptanalysis exists (as is typical for new ciphers), full-round Kalyna remains unbroken in practice and is intended for implementations that need the DSTU standard or the larger block/key options. 

---

# 5. Applications: Password storage, Proof of Work, Merkle Tree

### Password storage

One of the most important applications of hash functions in practice is **secure password storage**. Storing passwords in plaintext is extremely dangerous, since a database breach would expose all user credentials immediately. Instead, systems rely on **hashing** to protect stored passwords.

When a user creates an account, the system does not store the actual password. Instead, it applies a cryptographic hash function $h(\cdot)$ to the password $P$ and stores only the digest:

$$
D = h(P)
$$

Later, when the user attempts to log in, the system hashes the entered password and compares the digest with the stored value. If they match, the password is correct.

#### Challenges and Attacks

**Precomputed attacks (rainbow tables).** If a common hash function like SHA-256 is used without extra protection, attackers can precompute large tables of password-hash pairs and quickly recover weak passwords.

**Brute-force attacks.** Attackers can still try all possible passwords until one produces the correct hash. With fast modern hardware (e.g., GPUs, ASICs), billions of hashes can be computed per second.

#### Salting
To defend against precomputation and rainbow tables, systems introduce a **salt**:
- A random string $s$ is generated for each password.
- The hash is computed as $D = h(s \,\|\, P)$.
- Both $s$ and $D$ are stored in the database.

This ensures that even if two users choose the same password, their stored hashes will differ. It also makes rainbow tables useless, since the salt randomizes each hash computation.

#### Key Stretching
Even with salting, brute-force attacks remain possible if the hash function is very fast. To slow down attackers, password hashing schemes use **key stretching**: applying the hash function many times, or using functions specifically designed to be computationally expensive.

#### Password Hashing Algorithms
Modern systems do not use general-purpose hash functions like SHA-256 for password storage. Instead, they rely on specialized **password hashing functions**, such as:
- **PBKDF2 (Password-Based Key Derivation Function 2).** Repeated hashing with a salt, controlled by an iteration count.
- **bcrypt.** Based on the Blowfish cipher; incorporates salting and adjustable cost factors.
- **scrypt.** Memory-hard function, designed to resist attacks using GPUs/ASICs.
- **Argon2 (winner of the Password Hashing Competition, 2015).** Modern standard, with tunable time, memory, and parallelism parameters.

### Proof of Work 

Proof of Work (PoW) is a consensus mechanism used in many blockchain systems, most notably Bitcoin. Its main purpose is to secure the network and validate transactions without relying on a central authority.

In PoW, participants called **miners** compete to solve a computationally difficult problem. This problem involves finding a number, called a **nonce**, such that when it is combined with the block data and passed through a cryptographic hash function $H$, the output hash meets a specific condition, typically starting with a certain number of zeros:


$$
H(\text{block data} \, || \, \text{nonce}) < \text{target}
$$

The **target** is adjusted periodically to control the rate at which new blocks are added to the blockchain, maintaining a stable network operation.

Once a miner finds a valid nonce, the solution is broadcast to the network. Other nodes can easily verify the correctness of the solution by performing a single hash operation, which is computationally inexpensive. This ensures that blocks are valid and discourages malicious activity, as altering any transaction would require redoing the PoW for that block and all subsequent blocks—a prohibitively expensive task.

PoW provides three main benefits:

- Security. It makes attacks on the network costly and impractical.  
- Decentralization. No central authority is needed to validate transactions.  
- Consensus. All honest nodes can agree on the current state of the blockchain.  

Although PoW is effective, it requires significant computational resources and energy, which has led to the development of alternative mechanisms like Proof of Stake (PoS).


### Merkle tree

A **Merkle tree** (or **hash tree**) is a binary tree structure used to efficiently and securely verify the integrity of large data sets. It was introduced by Ralph Merkle in 1979 and is a fundamental building block in cryptography, blockchain technologies, and distributed systems.

![1](https://upload.wikimedia.org/wikipedia/commons/9/95/Hash_Tree.svg)

#### Structure

**Leaves**: Each leaf node contains a cryptographic hash of a data block.
**Internal nodes**: Each internal node is computed as the hash of the concatenation of its two children.
**Root**: The final single hash at the top of the tree is called the **Merkle root**.

Formally, let $h(\cdot)$ denote a cryptographic hash function:

- For leaves $L_i$, compute  

  $$H_i = h(L_i).$$  

- For internal nodes, compute recursively:

  $$H_{i,j} = h\big(H_i \, \| \, H_j\big),$$ 

  where $\|$ denotes concatenation.

The process continues until only one hash remains — the Merkle root.


#### Example

Suppose we have four data blocks $D_1, D_2, D_3, D_4$:

1. Compute leaf hashes: 

$$H_1 = h(D_1), \; H_2 = h(D_2), \; H_3 = h(D_3), \; H_4 = h(D_4).$$

2. Compute internal nodes: 

$$H_{12} = h(H_1 \| H_2), \quad H_{34} = h(H_3 \| H_4).$$

3. Compute root:  

$$R = h(H_{12} \| H_{34}).$$

Thus, $R$ is the Merkle root.

#### Advantages

**Efficient proofs**. To prove that a data block $D_i$ is included in the tree, only $ \approx \log n$ hashes are needed instead of all $n$ data blocks. A Merkle tree is a balanced binary tree of cryptographic hashes. To prove that a particular leaf (data block) $D_i$ is in the tree you only need the *authentication path* — the sibling hash at each level on the way from that leaf up to the root. A balanced binary tree of $n$ leaves has height about $\lceil\log_2 n\rceil$, so you need about $\lceil\log_2 n\rceil$ sibling hashes. That is exponential savings compared with sending all $n$ blocks.

**Tamper detection**. Any modification in a single leaf changes the Merkle root, making inconsistencies easy to detect.

Every internal node in a Merkle tree is the cryptographic hash of its children. If a single leaf value changes, its leaf hash changes and that change *propagates* up the tree: each parent hash that depends (directly or indirectly) on the modified leaf will also change. Therefore the final root hash will differ — except with negligible probability if the hash function is broken — so comparing roots detects any tampering.



#### Applications

**Data integrity verification**. Efficiently checks whether a specific piece of data is part of a larger dataset.

**Applications of Merkle Trees in NoSQL Systems**

Merkle trees are widely used in distributed databases and storage systems to ensure **data consistency** across replicas. A key challenge in such systems is to detect and reconcile differences between nodes efficiently, without having to exchange all the stored data. Merkle trees provide a compact way to summarize the contents of a dataset and to identify inconsistencies with logarithmic communication overhead.

**Blockchain**

Merkle trees are a fundamental component of blockchain systems like **Bitcoin** and **Ethereum**, where they are used to efficiently and securely summarize transactions within blocks. 

Each Bitcoin block contains a list of transactions. Instead of storing all transactions in the block header, Bitcoin computes a **Merkle root**, which is a single hash summarizing all transactions:

$$
R = \text{MerkleRoot}(\text{Tx}_1, \dots, \text{Tx}_n)
$$

This root is included in the block header and is used in proof-of-work mining and block validation.
 
Simplified Payment Verification (SPV) clients do not download full blocks. They verify that a transaction is included in a block using a **Merkle proof**, which contains only the hashes along the path from the transaction to the root (about $\log_2 n$ hashes for $n$ transactions).  
This allows lightweight clients to check inclusion without storing or transmitting the entire blockchain.
 
Any modification of a transaction in a block changes its leaf hash, which propagates to the Merkle root. If the computed root does not match the root in the block header, the block is invalid, ensuring data integrity.

**Benefits in Peer-to-Peer Networks**

1. **Efficient verification.** Nodes can verify inclusion of a transaction or state change with only a logarithmic number of hashes.
2. **Lightweight clients.** SPV clients (Bitcoin) and light Ethereum clients rely on Merkle proofs to interact securely with the blockchain without full data.
3. **Data integrity and tamper resistance.** Any modification in a block or account state is immediately detectable via the Merkle root.
4. **Bandwidth efficiency.** Nodes only exchange relevant hashes instead of entire datasets when synchronizing or verifying inclusion.