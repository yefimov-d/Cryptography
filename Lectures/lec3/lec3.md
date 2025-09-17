# Topic 3. Basic Principles of Block Ciphers

## Lecture Plan

1. Fundamentals of Block Cipher Operation  
2. Data Encryption Standard (DES)  
3. Triple DES (3DES)  
4. Advanced Encryption Standard (AES)  
5. National Standard of Ukraine – DSTU GOST 28147-2009  
6. National Standard of Ukraine – DSTU 7624:2014 (Kalyna)  

---

## 1. Fundamentals of Block Cipher Operation  

A block cipher is a symmetric key cryptographic algorithm that operates on fixed-size blocks of plaintext, transforming them into ciphertext using a secret key. Unlike stream ciphers, which encrypt data one bit or byte at a time, block ciphers encrypt data in discrete chunks, typically 64 or 128 bits.

### Operation Modes of Block Ciphers

Block ciphers encrypt fixed-size blocks, but most practical messages are longer than a single block. To securely encrypt longer messages, **modes of operation** define how to apply the block cipher repeatedly. Each mode has its properties, advantages, and potential vulnerabilities.

![1](https://upload.wikimedia.org/wikipedia/commons/e/ef/BlockCipherModesofOperation.svg)

**Electronic Codebook (ECB) Mode**
- **Description**: Each plaintext block is encrypted independently using the same key.
- **Encryption**: 

$$ C_i = E_K(P_i) $$

- **Decryption**:

$$ P_i = D_K(C_i) $$

- **Pros**: Simple, allows parallel encryption/decryption of blocks.

- **Cons**: Identical plaintext blocks produce identical ciphertext blocks → patterns in plaintext are visible in ciphertext. Not recommended for sensitive data.

**Cipher Block Chaining (CBC) Mode**

- **Description**: Each plaintext block is XORed with the previous ciphertext block before encryption.
- **Encryption**:

$$ C_0 = IV $$

$$ C_i = E_K(P_i \oplus C_{i-1}) $$

- **Decryption**:

$$ P_i = D_K(C_i) \oplus C_{i-1} $$

- **Pros**: Conceals patterns in plaintext; widely used.
- **Cons**: Sequential processing required; errors in one block propagate to the next block; requires a unique initialization vector (IV) for security.

**Propagating Cipher Block Chaining (PCBC) Mode**

- **Description**: PCBC is a variant of CBC designed to propagate both plaintext and ciphertext changes to subsequent blocks, increasing error diffusion. Each plaintext block is XORed with both the previous plaintext and ciphertext blocks before encryption. PCBC is mainly of historical or academic interest, illustrating a method to enhance diffusion in block cipher modes. 
- **Encryption**:

$$ C_0 = IV $$

$$ C_i = E_K(P_i \oplus P_{i-1} \oplus C_{i-1}) $$

- **Decryption**:

$$ P_i = D_K(C_i) \oplus P_{i-1} \oplus C_{i-1} $$

- **Pros**: A single-bit change in plaintext or ciphertext propagates to all subsequent blocks, improving error diffusion compared to CBC.  
- **Cons**: Like CBC, sequential processing is required. A transmission error in one block corrupts all following blocks, making it sensitive to errors. Not widely used in practice due to error propagation risks and limited standardization.  


**Cipher Feedback (CFB) Mode**
- **Description**: Converts a block cipher into a self-synchronizing stream cipher.
- **Encryption**:

$$ C_i = P_i \oplus E_K(C_{i-1}) $$

(with $C_0 = IV$)

- **Decryption**:

$$ P_i = C_i \oplus E_K(C_{i-1}) $$

- **Pros**: Can encrypt data smaller than block size; self-synchronizing if errors occur.
- **Cons**: Sequential processing required; IV must be unpredictable.

**Output Feedback (OFB) Mode**
- **Description**: Converts a block cipher into a synchronous stream cipher by generating a keystream.
- **Keystream Generation**:

$$ O_0 = IV $$

$$ O_i = E_K(O_{i-1}) $$

- **Encryption/Decryption**:

$$ C_i = P_i \oplus O_i $$

$$ P_i = C_i \oplus O_i $$

- **Pros**: Pre-computation of keystream possible; errors in transmission do not propagate.
- **Cons**: If IV repeats, security is compromised; sequential processing of keystream needed.

**Counter (CTR) Mode**
- **Description**: Each block is XORed with the encryption of a counter value. Allows parallel processing.
- **Encryption**:

$$ C_i = P_i \oplus E_K(Nonce \| Counter_i) $$

- **Decryption**:

$$ P_i = C_i \oplus E_K(Nonce \| Counter_i) $$

- **Pros**: Supports parallel encryption/decryption; random access possible; errors do not propagate.
- **Cons**: Counter/nonce must be unique for each encryption with the same key; otherwise, security is compromised.

**Summary**

| Mode | Parallelizable | Error Propagation | Suitable For |
|------|----------------|-----------------|--------------|
| ECB  | Yes            | No              | Rarely used  |
| CBC  | No             | Yes             | General purpose, file encryption |
| PCBC | No             | Yes (propagates errors further than CBC) | Systems needing strong error diffusion (historical/academic) |
| CFB  | No             | Limited         | Stream-like data |
| OFB  | No (keystream) | No              | Stream encryption, error-prone channels |
| CTR  | Yes            | No              | High-speed encryption, random access |


Choosing the correct mode is critical for both security and performance in cryptographic systems.

The famous example (see below) with the Linux penguin (“Tux”) image encrypted in ECB mode primarily demonstrates a limitation of determinism in block ciphers when no proper mode of operation is used. When the Tux image was encrypted with ECB mode, the output ciphertext still visibly resembled the original image. The repeating patterns of the image’s pixels produced repeating ciphertext blocks. Identical plaintext blocks produce identical ciphertext blocks. ECB does not sufficiently hide patterns in structured data. This shows why modern systems use CBC, CTR, or other modes with IVs/counters to break these patterns and ensure confidentiality.

| Original Image | Encrypted with ECB Mode | Encrypted with CTR Mode |
|----------------|-----------------------|------------------------|
| <img src="https://upload.wikimedia.org/wikipedia/commons/3/35/Tux.svg" width="150"/> | <img src="https://upload.wikimedia.org/wikipedia/commons/9/96/Tux_encrypted_ecb.png" width="150"/> | <img src="https://upload.wikimedia.org/wikipedia/commons/0/00/Tux_encrypted_ctr.png" width="150"/> |
| Original image of the Linux penguin. | ECB mode shows visible patterns due to determinism. | CTR mode appears randomized, hiding patterns. |

### National Standard of Ukraine — DSTU ISO/IEC 10116:2019

**DSTU ISO/IEC 10116:2019** is the Ukrainian national standard that specifies the **modes of operation for block ciphers**. It aligns with the international standard **ISO/IEC 10116:2019**, providing guidance on securely applying block ciphers to encrypt data of arbitrary length.

**Scope:** Defines secure methods for encrypting data using block ciphers  
**Purpose:** Ensures confidentiality by specifying standard modes of operation  
**Applicability:** Can be applied to any symmetric block cipher, such as AES or Kalyna  

**The standard includes well-known block cipher modes, such as ECB, CBC, CFB, OFB, CTR.**

While the underlying block cipher ensures the transformation of data blocks, **the mode of operation is critical** for achieving strong security in real-world applications, preventing issues such as pattern leakage and ensuring proper handling of variable-length data.

### Key Properties of Block Ciphers

**Deterministic**
A block cipher is deterministic in its basic form, meaning that for a given key and a specific plaintext block, the resulting ciphertext block is always identical.  
While determinism ensures consistent encryption, it can leak patterns if the same plaintext appears multiple times. This is why most practical applications use an **initialization vector (IV)** or **counter** to introduce randomness and prevent repeated ciphertext patterns.

**Reversible**
Block ciphers are reversible, which means there exists a decryption function that can perfectly recover the original plaintext from the ciphertext using the same key.  
This reversibility is crucial for symmetric encryption systems, allowing secure communication where both encryption and decryption rely on the same secret key. Any alteration in the ciphertext can disrupt decryption, emphasizing the need for integrity checks.

**Confusion and Diffusion**
The relationship between the key and the ciphertext is obscured by complex substitutions, making it difficult for an attacker to deduce the key even if they know some plaintext-ciphertext pairs.  
Each plaintext bit affects many ciphertext bits through permutations, spreading the influence of individual bits across the block. This ensures that small changes in plaintext or key produce significant changes in ciphertext, thwarting statistical attacks.  
The combination of confusion and diffusion is central to block cipher security, ensuring that ciphertext reveals minimal information about the original plaintext or key.

### Design
#### Feistel Network

A **Feistel network** is a fundamental design model used to build many symmetric block ciphers. It was first introduced by Horst Feistel and became the foundation of well-known encryption algorithms, such as **DES (Data Encryption Standard)**. The key idea is to divide the plaintext block into two halves and process them through multiple rounds of substitution and permutation using a round function.
  - Block size: $64$ bits
  - Key size: $56$ bits (effective)
  - $16$ Feistel rounds

Feistel networks provide a general blueprint for constructing secure ciphers. Even though newer algorithms like **AES** are not Feistel-based (they use a substitution–permutation network), Feistel designs remain important in understanding symmetric encryption.

![1](https://upload.wikimedia.org/wikipedia/commons/f/fa/Feistel_cipher_diagram_en.svg)


**Structure**

1. **Initial Split**
The input block of size $2n$ bits is divided into two halves:
- Left half: $L_0$
- Right half: $R_0$

2. **Round Function** 
For each round $i$, the following transformations are applied:

$$
L_i = R_{i-1}
$$

$$
R_i = L_{i-1} \oplus F(R_{i-1}, K_i)
$$

Here:
  - $F$ is the round function (can be complex and include substitution, permutation, and key mixing).
  - $K_i$ is the subkey derived from the main secret key.

3. **Final Swap (Optional)** 
After the last round, the halves may be swapped or left as is, depending on the cipher’s specification.

**Properties**
Inversion. Decryption uses the same structure as encryption. The only difference is that the subkeys are applied in reverse order.

Flexibility. The round function $F$ does not need to be invertible for the overall cipher to be reversible.

Security. The security of the Feistel network depends on the number of rounds and the design of the round function.


#### Substitution–permutation network

A **Substitution–Permutation Network (SPN)** is a fundamental structure used in the design of modern block ciphers. It provides both **confusion** and **diffusion**, two essential properties for secure encryption, as described by Claude Shannon.


<img src="https://upload.wikimedia.org/wikipedia/commons/a/ab/SubstitutionPermutationNetwork-en.svg" alt="Substitution–Permutation Network" width="400"/>

**Key Concepts**

**Substitution (S-boxes)**. Non-linear transformation of input bits. Introduces *confusion* by making the relationship between the key and ciphertext complex. Implemented using lookup tables or mathematical functions.

**Permutation (P-boxes)**. Linear transformation that rearranges bits across the block.  
Provides *diffusion* by spreading the influence of each input bit across many output bits.  

**Rounds**. An SPN applies several rounds of alternating substitution and permutation. Each round usually includes key mixing (XOR with round key), substitution, and permutation.

**Structure of an SPN**

1. **Key mixing** – Combine the plaintext block with a round key using XOR.  
2. **Substitution** – Apply S-boxes to groups of bits.  
3. **Permutation** – Apply P-box to shuffle bits.  
4. **Repeat** – Several rounds ensure strong security.  
5. **Final key mixing** – Often, an additional key mixing is done at the end.

The **Advanced Encryption Standard (AES)** is a well-known block cipher that uses an SPN-like design, with S-boxes for substitution and a *ShiftRows*/*MixColumns* step for permutation.

**Advantages of SPNs**

- Strong security with enough rounds.  
- Well-suited for both software and hardware implementations.  
- Flexible design: can be adapted to different block and key sizes.

---

## 2. Data Encryption Standard (DES)

The Data Encryption Standard (DES) was developed by IBM in the 1970s and officially adopted as a federal encryption standard in 1977 by NIST (then the National Bureau of Standards).  

DES played a foundational role in the development of symmetric-key cryptography. It popularized the concept of block ciphers and influenced later standards and research in cryptanalysis, including the development of Triple DES (3DES) and AES.  

DES was widely used in securing financial transactions (e.g., in ATM systems, banking networks), telecommunications, and early computer systems. Its use declined as computational power made brute-force attacks feasible, leading to its replacement by AES in 2001.  

**Basic Characteristics**  
- Block cipher with 64-bit block size  
- Key length: 56 effective bits (plus 8 parity bits)  
- Symmetric-key encryption: same key used for both encryption and decryption  

**High-Level Structure**  

DES follows a **Feistel network** structure with three main stages:  

1. **Initial Permutation (IP)**  
The plaintext block of 64 bits is permuted according to a fixed table.  This step does not add security by itself but helps simplify hardware implementation. After permutation, the block is split into two halves: left (32 bits) and right (32 bits).  

2. **16 Feistel Rounds**  
Each round applies a Feistel function `F` to the right half and mixes it with the left half.  
Operations inside each round:  
     - **Expansion (E):** The 32-bit right half is expanded to 48 bits.  
     - **Key mixing:** A 48-bit round key (derived from the main key) is XORed with the expanded block.  
     - **Substitution (S-boxes):** The 48-bit result is passed through 8 S-boxes, reducing it back to 32 bits.  
     - **Permutation (P):** The 32-bit output is permuted to further diffuse the bits.  
   - The output of `F` is XORed with the left half, then halves are swapped for the next round.  

3. **Final Permutation (Inverse IP)**  
After 16 rounds, the left and right halves are combined.  
A final permutation, which is the inverse of the Initial Permutation (IP), is applied.  
This produces the final 64-bit ciphertext block.  

![1](https://upload.wikimedia.org/wikipedia/commons/2/25/Data_Encription_Standard_Flow_Diagram.svg)


**Feistel Round Details**  

A single round of DES operates on two halves of the data block: the **Left half (L)** and the **Right half (R)**, each of 32 bits.  

1. **Splitting into halves**  
At round $i$, the block is represented as $(L_{i-1}, R_{i-1})$.  

2. **Expansion function (32 → 48 bits)**  
The right half $R_{i-1}$ is expanded from 32 to 48 bits using a fixed expansion table:  

$$
E : R_{i-1} \in \{0,1\}^{32} \;\;\to\;\; \{0,1\}^{48}
$$  

3. **Key mixing (round key XOR)**  
A 48-bit round key $K_i$ is generated from the main key schedule. The expanded right half is XORed with the round key:

$$
B = E(R_{i-1}) \oplus K_i
$$  

4. **Substitution (S-boxes)**  
The 48-bit value $B$ is divided into eight 6-bit chunks.  
Each chunk is passed through an **S-box** $S_j$, producing a 4-bit output. After all 8 S-boxes: 

$$
S(B) \in \{0,1\}^{32}
$$  

5. **Permutation (P-box)**  
The 32-bit output of the S-boxes is permuted to spread (diffuse) the bits:  

$$
P: \{0,1\}^{32} \;\;\to\;\; \{0,1\}^{32}
$$  

6. **Feistel function result**  
The round function $F$ is:  

$$
F(R_{i-1}, K_i) = P(S(E(R_{i-1}) \oplus K_i))
$$  

7. **Swap of halves**  
The new halves are computed as: 

$$
L_i = R_{i-1}
$$  

$$
R_i = L_{i-1} \oplus F(R_{i-1}, K_i)
$$  

Thus, after each round, the halves are swapped and the process repeats for 16 rounds.  

**Key Schedule**  

The DES key schedule generates 16 round keys ($K_1, K_2, \dots, K_{16}$), each 48 bits long, from the original 64-bit user-supplied key.  

1. **56-bit key (dropping parity bits)**  
The original 64-bit key includes 8 parity bits (1 per byte).  
After removing parity bits, the effective key size is 56 bits.  

2. **Permutation Choice 1 (PC-1)**  
The 56-bit key is permuted and split into two halves of 28 bits each:  

$$
(C_0, D_0) = PC1(\text{Key})
$$  

3. **Rotations (Shifts)**  
In each round, both $C_{i-1}$ and $D_{i-1}$ are circularly left-shifted by 1 or 2 positions (depending on the round).  
This produces:  

$$
C_i = \text{LeftShift}(C_{i-1}) \\
D_i = \text{LeftShift}(D_{i-1})
$$  

4. **Permutation Choice 2 (PC-2)**  
The 56-bit concatenation $(C_i, D_i)$ is permuted and reduced to 48 bits to form the round key $K_i$:  

$$
K_i = PC2(C_i \| D_i)
$$  

5. **Importance of key schedule**  
It ensures that round keys are different and uniformly distributed. Key schedule prevents weak keys (e.g., identical subkeys in multiple rounds). Increases resistance against attacks such as differential and linear cryptanalysis.  

**Decryption Process**  

DES decryption uses the **same Feistel network structure** as encryption.  The only difference is the **order of round keys**: during decryption, the 16 round keys are applied in reverse order ($K_{16}, K_{15}, \dots, K_{1}$).  Because of the Feistel design, the operations (expansion, XOR, substitution, permutation) remain unchanged.  This symmetry simplifies implementation: the same hardware or software routine can perform both encryption and decryption by just reversing the key schedule.  

**Security Considerations**  

When introduced, DES was considered secure, but **56-bit keys are too short by modern standards**.  
**Brute-force attack**. All $2^{56}$ keys can be tested, which is feasible with modern computing power.   
**Historical break**. In 1998, the EFF built the *DES Cracker* machine, capable of finding a DES key in less than 3 days.  


**Practical Example**  

Encrypting a 64-bit block with DES using Python (`pycryptodome`):  

```python
from Crypto.Cipher import DES
from Crypto.Random import get_random_bytes

# Generate a random 8-byte (64-bit) DES key
key = get_random_bytes(8)  

# Create DES cipher in ECB mode
cipher = DES.new(key, DES.MODE_ECB)

# Example plaintext (must be multiple of 8 bytes)
plaintext = b"OpenAI42"  

# Encrypt
ciphertext = cipher.encrypt(plaintext)

# Decrypt
decrypted = cipher.decrypt(ciphertext)

print("Key:", key.hex())
print("Plaintext:", plaintext)
print("Ciphertext:", ciphertext.hex())
print("Decrypted:", decrypted)
```

---

## 3. Triple DES (3DES)

**Triple DES (3DES)** is a symmetric-key block cipher which applies the **Data Encryption Standard (DES)** algorithm three times to each data block. It was developed to provide a more secure alternative to DES, which became vulnerable to brute-force attacks due to its short 56-bit key size.

**Block size:** 64 bits  
**Key sizes:** 112 bits (two-key 3DES) or 168 bits (three-key 3DES)  
**Rounds:** 48 internal DES rounds (16 per DES operation × 3)  
**Mode:** Encrypt-Decrypt-Encrypt (EDE) or Encrypt-Encrypt-Encrypt (EEE)  

**EDE Mode (Encrypt-Decrypt-Encrypt)**
The most common mode of 3DES is EDE, which involves three stages:

1. **Encrypt** with key $K_1$  
2. **Decrypt** with key $K_2$  
3. **Encrypt** with key $K_3$  

Formally, if $P$ is the plaintext and $C$ is the ciphertext:

$$
C = E_{K_3}(D_{K_2}(E_{K_1}(P)))
$$

For decryption, the order of keys is reversed:

$$
P = D_{K_1}(E_{K_2}(D_{K_3}(C)))
$$

> If $K_1 = K_2 = K_3$, 3DES is equivalent to regular DES.

**EEE Mode (Encrypt-Encrypt-Encrypt)**
Less common, all three DES operations are encryption steps:

$$
C = E_{K_3}(E_{K_2}(E_{K_1}(P)))
$$

#### Advantages

- Stronger security than single DES  
- Compatible with legacy DES systems  
- Can be used with two keys (112-bit security) or three keys (168-bit security)

#### Disadvantages

- **Slower** than DES due to triple operations  
- Vulnerable to **meet-in-the-middle attacks**  
- Limited block size (64-bit) makes it susceptible to **birthday attacks** on large datasets


3DES extends DES by applying it three times in sequence with either two or three keys. While more secure than DES, it is largely replaced today by **AES**, which offers better security and performance.

---

## 4. Advanced Encryption Standard (AES) 

The **Advanced Encryption Standard (AES)** is a symmetric block cipher that has become the global standard for secure data encryption. It was established by the U.S. National Institute of Standards and Technology (NIST) in 2001 as the successor to the older **Data Encryption Standard (DES)**. The main reason for developing AES was the discovery of **security vulnerabilities in DES**, especially its relatively short key length (56 bits), which made it susceptible to brute-force attacks as computing power grew.

AES was designed to provide stronger security, efficiency, and flexibility. It supports key sizes of **128, 192, and 256 bits**, making it resistant to known attack methods while also being efficient in both hardware and software implementations. Unlike DES, which had become outdated, AES was built to withstand modern cryptanalytic techniques and to remain secure for decades.

Today, AES plays a **central role in modern cryptography**. It is used in a wide range of applications and protocols, including:

- **Internet security protocols** (TLS/SSL for HTTPS websites)
- **Wireless communication** (WPA2/WPA3 for Wi-Fi encryption)
- **Virtual Private Networks (VPNs)**
- **File and disk encryption tools**
- **Government and military data protection**

Because of its balance of strong security and high performance, AES is trusted worldwide as the foundation of secure communication and data protection in the digital age.

### AES Overview

**Block Cipher Characteristics**
AES is a **block cipher**, which means it encrypts fixed-size blocks of data at a time.  

|  | Key Length *(Nk words)* | Block Size *(Nb words)* | Number of Rounds *(Nr)* |
|----------|--------------------------|--------------------------|--------------------------|
| **AES-128** | 4                        | 4                        | 10                       |
| **AES-192** | 6                        | 4                        | 12                       |
| **AES-256** | 8                        | 4                        | 14                       |


Each round applies a series of transformations to the data, ensuring strong diffusion and confusion, which are essential properties of secure encryption.


**Design**
AES is based on the **Substitution-Permutation Network (SPN)** design. This structure consists of multiple rounds of transformations that combine substitution (replacing bytes with others using an S-box) and permutation (rearranging bits or bytes).  
The SPN design ensures that small changes in the plaintext or key produce completely different ciphertext outputs, making AES resistant to common cryptanalysis techniques.


#### AES Components

AES (Advanced Encryption Standard) is built from several core transformations applied in multiple rounds. Each component plays a role in ensuring **confusion** and **diffusion**, two fundamental cryptographic principles.



**1. SubBytes**
Each byte in the state matrix is replaced using a substitution box (S-box). The S-box is derived from multiplicative inverses in $F_{2^8}$, followed by an affine transformation. **SubBytes** introduces non-linearity and prevents simple algebraic attacks.

![1](https://upload.wikimedia.org/wikipedia/commons/a/a4/AES-SubBytes.svg)

The AES S-box is not chosen randomly — it is generated using a mathematical process to ensure cryptographic strength. Each byte (except `00`) is replaced by its multiplicative inverse in the finite AES field $F_{2^8}$.  The result is then transformed using a fixed **affine transformation** over $F_{2^8}$. Each bit of the output is computed as a linear combination of bits of the inverse, followed by adding a constant vector.


**AES S-box table** (for example, ``01`` → ``7c``):

|   | 0  | 1  | 2  | 3  | 4  | 5  | 6  | 7  | 8  | 9  | a  | b  | c  | d  | e  | f  |
|---|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| 0 | 63 | 7c | 77 | 7b | f2 | 6b | 6f | c5 | 30 | 01 | 67 | 2b | fe | d7 | ab | 76 |
| 1 | ca | 82 | c9 | 7d | fa | 59 | 47 | f0 | ad | d4 | a2 | af | 9c | a4 | 72 | c0 |
| 2 | b7 | fd | 93 | 26 | 36 | 3f | f7 | cc | 34 | a5 | e5 | f1 | 71 | d8 | 31 | 15 |
| 3 | 04 | c7 | 23 | c3 | 18 | 96 | 05 | 9a | 07 | 12 | 80 | e2 | eb | 27 | b2 | 75 |
| 4 | 09 | 83 | 2c | 1a | 1b | 6e | 5a | a0 | 52 | 3b | d6 | b3 | 29 | e3 | 2f | 84 |
| 5 | 53 | d1 | 00 | ed | 20 | fc | b1 | 5b | 6a | cb | be | 39 | 4a | 4c | 58 | cf |
| 6 | d0 | ef | aa | fb | 43 | 4d | 33 | 85 | 45 | f9 | 02 | 7f | 50 | 3c | 9f | a8 |
| 7 | 51 | a3 | 40 | 8f | 92 | 9d | 38 | f5 | bc | b6 | da | 21 | 10 | ff | f3 | d2 |
| 8 | cd | 0c | 13 | ec | 5f | 97 | 44 | 17 | c4 | a7 | 7e | 3d | 64 | 5d | 19 | 73 |
| 9 | 60 | 81 | 4f | dc | 22 | 2a | 90 | 88 | 46 | ee | b8 | 14 | de | 5e | 0b | db |
| a | e0 | 32 | 3a | 0a | 49 | 06 | 24 | 5c | c2 | d3 | ac | 62 | 91 | 95 | e4 | 79 |
| b | e7 | c8 | 37 | 6d | 8d | d5 | 4e | a9 | 6c | 56 | f4 | ea | 65 | 7a | ae | 08 |
| c | ba | 78 | 25 | 2e | 1c | a6 | b4 | c6 | e8 | dd | 74 | 1f | 4b | bd | 8b | 8a |
| d | 70 | 3e | b5 | 66 | 48 | 03 | f6 | 0e | 61 | 35 | 57 | b9 | 86 | c1 | 1d | 9e |
| e | e1 | f8 | 98 | 11 | 69 | d9 | 8e | 94 | 9b | 1e | 87 | e9 | ce | 55 | 28 | df |
| f | 8c | a1 | 89 | 0d | bf | e6 | 42 | 68 | 41 | 99 | 2d | 0f | b0 | 54 | bb | 16 |

See also: https://en.wikipedia.org/wiki/Rijndael_S-box

**2. ShiftRows**
Cyclically shifts rows of the state matrix.
  - Row 0: no shift  
  - Row 1: shift left by 1 byte  
  - Row 2: shift left by 2 bytes  
  - Row 3: shift left by 3 bytes
**Purpose**: Provides inter-column diffusion.

![1](https://upload.wikimedia.org/wikipedia/commons/6/66/AES-ShiftRows.svg)

**3. MixColumns**
Each column of the state is treated as a polynomial over $F_{2^8}$ and multiplied modulo $x^4 + 1$ with a fixed polynomial:

$$
c(x) = 03x^3 + 01x^2 + 01x + 02
$$

Matrix form:

$$
\begin{bmatrix}
b_0' \\
b_1' \\
b_2' \\
b_3'
\end{bmatrix}
\coloneqq
\begin{bmatrix}
02 & 03 & 01 & 01 \\
01 & 02 & 03 & 01 \\
01 & 01 & 02 & 03 \\
03 & 01 & 01 & 02
\end{bmatrix}
\cdot
\begin{bmatrix}
b_0 \\
b_1 \\
b_2 \\
b_3
\end{bmatrix}
$$

**Purpose**: Ensures mixing of bytes within a column (diffusion).

![1](https://upload.wikimedia.org/wikipedia/commons/7/76/AES-MixColumns.svg)


**4. AddRoundKey**
XOR of the state matrix with the round key:

  $$
  S' = S \oplus K_{\text{round}}
  $$

**Purpose**: Directly introduces key material into the cipher, ensuring security depends on the secret key.

![1](https://upload.wikimedia.org/wikipedia/commons/a/ad/AES-AddRoundKey.svg)

**5. Key Expansion**
The original key is expanded into multiple round keys using:
- Byte substitution with S-box  
- Rotations of 4-byte words  
- XOR with round constants (Rcon)

**AES encryption process**

Pseudocode:
```
Input: Plaintext P, Key schedule {K0, K1, ..., KNr}
Output: Ciphertext C

// Initial Round
S = P XOR K0

// Main Rounds
for i = 1 to Nr-1 do
    S = SubBytes(S)
    S = ShiftRows(S)
    S = MixColumns(S)
    S = S XOR Ki
end for

// Final Round
S = SubBytes(S)
S = ShiftRows(S)
C = S XOR KNr
```

**AES Key Schedule**

The **AES Key Schedule** defines how the original secret key is expanded into multiple round keys used in each round of encryption and decryption. It ensures that each round uses a unique key derived from the original key, providing security against attacks that exploit repeated key material. Purpose:
AES Key Schedule ensures each round has a **unique key** derived from the secret key, provides **security against key-recovery attacks**. The **same key schedule** to be reused for both encryption and decryption efficiently.

Consider the proccess closely.

The original key is expanded into an array of words, $W[0 \dots N_b(N_r+1)-1]$, where:
$N_b$: number of 32-bit words in a block (4 for AES-128)  
$N_r$: number of rounds (10, 12, 14 depending on key size)  

**Process**:
1. **Rotation (RotWord)**: Cyclically rotate a 4-byte word.  
2. **Substitution (SubWord)**: Apply S-box to each byte.  
3. **Round constant (Rcon) addition**: XOR with a constant unique to each round.  
4. **XOR with previous words**: Combine with previous words to generate new words.
**Formula**:

$$
W[i] = W[i-N_k] \oplus \text{SubWord}(\text{RotWord}(W[i-1])) \oplus Rcon[i/N_k]
$$

for $i \ge N_k$, where $N_k$ is the key length in words.



#### AES Decryption

AES decryption is the reverse process of encryption. It applies the **inverse operations** in reverse order to recover the original plaintext from ciphertext. Importantly, the same round keys generated during encryption are used, but in reverse order.



1. **InvSubBytes**  
Each byte is substituted using the inverse S-box (see [here](https://en.wikipedia.org/wiki/Rijndael_S-box)).  
Reverses the non-linear substitution of SubBytes.  

2. **InvShiftRows**  
Rows are cyclically shifted in the opposite direction:  
Row 1: shift right by 1 byte  
Row 2: shift right by 2 bytes  
Row 3: shift right by 3 bytes  
Reverses the permutation of ShiftRows.  

3. **InvMixColumns**  
Each column is multiplied by the inverse matrix over AES field: 

$$
\begin{bmatrix}
0E & 0B & 0D & 09 \\
09 & 0E & 0B & 0D \\
0D & 09 & 0E & 0B \\
0B & 0D & 09 & 0E
\end{bmatrix}
\cdot
\begin{bmatrix}
b_0 \\
b_1 \\
b_2 \\
b_3
\end{bmatrix}
$$

Reverses the diffusion introduced by MixColumns.  

4. **AddRoundKey**  
Identical to encryption: 

$$
S' = S \oplus K_{\text{round}}
$$

Since XOR is its own inverse, this step is unchanged.

**Key Schedule Reversal**

AES uses the same expanded key for encryption and decryption, but **in reverse order**:

Decryption: $K_{N_r}, K_{N_r-1}, \dots, K_0$  

Decryption rounds apply:
  - Initial round: **AddRoundKey** with $K_{N_r}$  
  - Main rounds: **InvShiftRows → InvSubBytes → AddRoundKey → InvMixColumns**  
  - Final round: **InvShiftRows → InvSubBytes → AddRoundKey** (without InvMixColumns)

#### Security and Real-World Applications

AES is designed to be highly secure against various attacks. Brute-force due to large key sizes (128, 192, 256 bits). Linear and differential cryptanalysis thanks to strong non-linear and diffusion layers. AES has no known practical attacks on full-round versions, making it reliable for modern cryptography. 

AES is widely used due to its efficiency and strong security:

- **Protocol Integration**: Secures data in transit via **TLS/SSL**, **IPsec**, and **WPA2** for Wi-Fi.  
- **File and Disk Encryption**: Used in tools like **BitLocker**, **VeraCrypt**, and other software for protecting sensitive data.  
- **Hardware Acceleration**: Modern CPUs support **AES-NI** instructions, speeding up encryption and decryption without extra software overhead.  

AES provides a balance of security and performance, making it a standard choice for both software and hardware encryption.

---

# 5. National Standard of Ukraine – DSTU GOST 28147-2009


**DSTU GOST 28147:2009** is the national standard of Ukraine that defines a cryptographic transformation algorithm for information processing systems in electronic computing networks. This standard is identical to the old standard GOST 28147-89 and regulates the rules for data encryption and MAC (Message Authentication Code) generation. 

Although GOST 28147-89 (and its Ukrainian counterpart DSTU GOST 28147:2009) provides a certain level of cryptographic protection, modern information security requirements demand more robust algorithms, such as AES or the "Kalyna" algorithm.

**Block size:** 64 bits
**Key size:** 256 bits
**Number of rounds:** 32
**Cipher type:** Block cipher based on the Feistel network
**Operation modes:**
- Simple substitution
- Stream encryption
- Stream encryption with feedback
- MAC generation


The algorithm includes:

**Key storage unit (KSU):** 256 bits, divided into eight 32-bit elements
**Four 32-bit accumulators:** for data storage
**Two 32-bit accumulators:** for storing constant values C1 and C2

---

# 6. National Standard of Ukraine – DSTU 7624:2014 (Kalyna) 

**DSTU 7624:2014**, also known as the **Kalyna algorithm**, is the national standard of Ukraine for symmetric key block encryption. It was developed to provide a modern, secure alternative to older standards like DSTU GOST 28147:2009, meeting contemporary information security requirements.


**Block size:** 128, 256, or 512 bits  
**Key size:** 128, 256, or 512 bits  
**Number of rounds:** Depends on key and block size 
**Cipher type:** Block cipher based on substitution-permutation network (SPN)  
**Security level:** Designed to meet modern cryptographic security requirements and resistant to known attacks on older standards

Rounds ($N_k$ is the key size and $N_b$ is the block size):

| $N_k$ \ $N_b$         | $N_b = 2$ (128 bits) | $N_b = 4$ (256 bits) | $N_b = 8$ (512 bits) |
| --------------------- | --------------------- | --------------------- | --------------------- |
| $N_k = 2$ (128 bits) | 10                    | 14                    | –                     |
| $N_k = 4$ (256 bits) | –                     | 14                    | 18                    |
| $N_k = 8$ (512 bits) | –                     | –                     | 18                    |

Kalyna uses:

**Substitution (S-box) transformations:** For non-linear confusion  
**Permutation layers:** To provide diffusion across the block  
**Key scheduling:** Generates round keys from the main key using a combination of modular arithmetic and substitution-permutation operations  

Kalyna is considered the primary symmetric encryption standard in Ukraine and serves as a replacement for DSTU GOST 28147:2009 in modern secure systems. Kalyna was designed to comply with international security standards while addressing the limitations of older block ciphers like GOST 28147-89. 