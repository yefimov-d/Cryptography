# Topic 4. Stream Ciphers

## Lecture Plan

1. Pseudorandom Number Generators in Cryptography
2. Fundamentals of Stream Ciphers
3. RC4
4. Salsa20
5. Strumok — Ukrainian Stream Cipher (DSTU 8845:2019)
---

## 1. Pseudorandom Number Generators in Cryptography 

### Definitions and cryptographic requirements
A **pseudorandom number generator (PRNG)** is an algorithm that expands a short, truly random seed into a long stream of bits that *appear* random. A **cryptographically secure PRNG (CSPRNG)** must additionally satisfy strong security properties so its outputs can safely be used for cryptographic purposes.

Key requirements for CSPRNGs:

**Unpredictability.** Given all previous outputs, an attacker cannot predict future outputs (forward secrecy).  

**Backtracking resistance.** Compromise of internal state should not reveal past outputs (backward secrecy) unless seed/state prior to compromise is known.  

**Uniformity / statistical soundness.** Output should pass standard randomness tests (but passing tests alone ≠ cryptographic security).  

**Sufficient period and throughput** for intended use.  

**Proper seeding / reseeding** from high-entropy sources.

### Entropy and entropy sources

**Entropy** is true randomness collected from physical events (hardware RNGs, user input timing, network jitter, thermal noise). Seed quality is critical: a CSPRNG with a low-entropy seed is insecure. Systems should gather entropy from multiple sources and reseed periodically (or on detected events).

Common OS-provided entropy sources: `/dev/random`, `/dev/urandom` (Unix-like), OS APIs (Windows CryptoAPI / CNG), and high-level language wrappers (`secrets` in Python).


### Classes of pseudorandom generators

**Deterministic PRNGs** (algorithmic, seed → stream). If not designed for crypto, they are predictable given state.  

**CSPRNGs / DRBGs** — deterministic but designed with cryptographic primitives (block ciphers in counter mode, stream ciphers, hash/HMAC based). Examples: AES-CTR-DRBG, HMAC-DRBG, Hash-DRBG, ChaCha20-based DRBG. 

**True/Hardware RNGs (TRNG / HRNG):** not deterministic; provide entropy which seeds DRBGs.

**Hybrid designs** (TRNG seeds a DRBG; DRBG supplies fast outputs).


### Non-cryptographic generators (examples & pitfalls)
These are fast and useful for simulations but **not safe** for cryptographic use.

**Linear Congruential Generator (LCG)**
Definition: $X_{n+1} = (a \cdot X_n + c) \mod m$.

```python
def lcg(seed, a=1664525, c=1013904223, m=2**32):
    x = seed
    while True:
        x = (a * x + c) % m
        yield x

gen = lcg(42)
numbers = [next(gen) for _ in range(10)]
print(numbers)
```

**Pitfalls:** small state, linear structure ⇒ easy to predict (observe a few outputs, solve for `a`, `c`, seed).



**Mersenne Twister**
Used by many standard library ```random``` in Python. Very good statistical properties for simulations but not cryptographically secure: internal state can be recovered from outputs.
Never use random / Mersenne Twister / LCG / xorshift for keys, nonces, or secrets.

**Not cryptographically secure**: if enough output values are observed, the internal state can be reconstructed, allowing prediction of all future values.


### OS Cryptographically Secure Pseudorandom Number Generators (CSPRNG)

Modern operating systems provide built-in **Cryptographically Secure Pseudorandom Number Generators (CSPRNGs)**. These are essential for generating high-quality random values suitable for security applications such as cryptographic keys, nonces, and tokens.

**Common OS CSPRNGs**

**`/dev/urandom` (Unix/Linux)**  
  A special file in Unix-like systems that provides cryptographically secure random bytes. It gathers environmental noise from device drivers and system activity.

**`CryptGenRandom` (Windows)**  
A Windows API function (legacy, now replaced by `BCryptGenRandom`) that provides cryptographically secure random data for applications.

**`getrandom` (Linux)**  
A newer system call that provides secure random bytes directly from the kernel without requiring access to `/dev/urandom`. It ensures randomness is available without blocking indefinitely.

**`secrets` Module (Python)**  
Python’s `secrets` module offers a high-level interface for generating secure tokens and keys. Internally, it uses the operating system’s CSPRNG.

**Example in Python**

```python
import secrets

# Generate a 256-bit random key (32 bytes)
key = secrets.token_bytes(32)
print(key.hex())
```

**Block Cipher in Counter Mode (AES-CTR DRBG)**

A Deterministic Random Bit Generator (DRBG) is a pseudorandom number generator designed for cryptographic use. One common construction is based on a **block cipher in counter (CTR) mode**, typically using the **Advanced Encryption Standard (AES)**. This design is known as **AES-CTR DRBG**.

**How Does AES-CTR DRBG Work**

1. **Initialization**  
   - The DRBG is seeded with an entropy input (random seed) and optional personalization string.  
   - This creates an internal **key** and **counter (V)**.

2. **Generation**  
   - To generate random output, the block cipher (AES) encrypts the counter value.  
   - The counter is incremented for each block.  
   - The concatenation of these encrypted blocks forms the pseudorandom output.

   Mathematically: 

$$
R_i = AES_K(V + i)
$$

where:  
- $K$ is the secret AES key,  
- $V$ is the counter value,  
- $i$ is the block index,  
- $R_i$ is the output block.

3. **Reseeding**  
   - Periodically, the DRBG is reseeded with new entropy to maintain security.

## Properties

- **Cryptographic strength**: Security relies on AES being a strong block cipher.  
- **Forward security**: If the state is compromised, past outputs remain secure.  
- **Scalability**: Can generate large amounts of pseudorandom data efficiently.  
- **Standardized**: Specified in [NIST SP 800-90A Rev. 1](https://csrc.nist.gov/publications/detail/sp/800-90a/rev-1/final).

**Example**

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

# Initialization
key = os.urandom(32)        # 256-bit AES key
counter = int.from_bytes(os.urandom(16), "big")

cipher = Cipher(algorithms.AES(key), modes.ECB())
encryptor = cipher.encryptor()

# Generate 5 pseudorandom blocks
random_blocks = []
for i in range(5):
    block_input = (counter + i).to_bytes(16, "big")
    random_blocks.append(encryptor.update(block_input))

random_data = b"".join(random_blocks)
print(random_data.hex())
```

**Other CSPRNGs**

**ChaCha20 / Salsa-based generators**  
ChaCha20 and Salsa20 are stream ciphers designed for both performance and security. When used as pseudorandom number generators, they take a secret key and nonce, then generate a keystream of pseudorandom bytes. Their main advantages are high speed on software implementations, resistance to cryptanalysis, and availability in modern cryptographic libraries. ChaCha20 in particular is widely deployed in TLS, SSH, and operating system randomness APIs.

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

key = os.urandom(32)       # 256-bit key
nonce = os.urandom(16)     # 128-bit nonce

cipher = Cipher(algorithms.ChaCha20(key, nonce), mode=None)
encryptor = cipher.encryptor()

# Generate 64 bytes of pseudorandom data
random_data = encryptor.update(b'\x00' * 64)
print(random_data.hex())
```

**HMAC-DRBG / Hash-DRBG (NIST SP 800-90A family)**  
These generators are standardized by NIST as part of the SP 800-90A specification. They rely on secure hash functions (like SHA-256) or keyed constructions (HMAC) to produce pseudorandom bits. Their design ensures reproducibility (determinism) while maintaining security properties such as forward secrecy and backtracking resistance. The difference lies in the primitive used: Hash-DRBG uses iterative hashing, while HMAC-DRBG uses a keyed hash for stronger guarantees.

```python
import hmac
import hashlib

def hmac_drbg(seed, n_bytes):
    v = b'\x01' * 32
    k = b'\x00' * 32
    result = b''
    while len(result) < n_bytes:
        k = hmac.new(k, v + b'\x00' + seed, hashlib.sha256).digest()
        v = hmac.new(k, v, hashlib.sha256).digest()
        result += v
    return result[:n_bytes]

seed = b'some entropy input'
random_bytes = hmac_drbg(seed, 32)
print(random_bytes.hex())

```

**Fortuna**  
Fortuna is a cryptographically secure PRNG proposed by Bruce Schneier and Niels Ferguson. It improves upon earlier designs (like Yarrow) by introducing multiple entropy pools. Incoming entropy is distributed across pools, and reseeding occurs progressively from them, making it resistant to state compromise. For output generation, Fortuna typically uses AES in counter mode. The design emphasizes robustness, simplicity, and resistance to attacks even if some entropy sources are compromised.

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

class SimpleFortuna:
    def __init__(self, key=None):
        self.key = key or get_random_bytes(32)
        self.counter = int.from_bytes(get_random_bytes(16), 'big')
    
    def generate(self, n_bytes):
        cipher = AES.new(self.key, AES.MODE_ECB)
        output = b''
        for _ in range((n_bytes + 15) // 16):
            block = cipher.encrypt(self.counter.to_bytes(16, 'big'))
            output += block
            self.counter += 1
        return output[:n_bytes]

fortuna = SimpleFortuna()
print(fortuna.generate(32).hex())
```

**Blum Blum Shub (BBS)**  
**Blum Blum Shub (BBS)** is a cryptographically secure pseudorandom number generator (CSPRNG), proposed in 1986 by Lenore Blum, Manuel Blum, and Michael Shub. Its security relies on the hardness of integer factorization.

BBS is based on quadratic residues. 

Let's choose two large prime numbers $p$ and $q$, both congruent to $3 \pmod{4}$:

$$
p \equiv 3 \pmod{4}, \quad q \equiv 3 \pmod{4}
$$

Then compute their product:

$$
M = p \cdot q
$$

The number $M$ is called the **Blum integer**.

A seed $s$ is selected such that:

$$
\gcd(s, M) = 1.
$$

Sequence Generation:

1. Initialize:

$$
x_0 = s^2 \pmod{M}
$$

2. For each step $i \geq 1$:

$$
x_i = x_{i-1}^2 \pmod{M}
$$

   The sequence $(x_i)$ is thus generated iteratively.

3. Extract output bits. The most common choice is the least significant bit (LSB):

$$
b_i = x_i \pmod{2}
$$

This produces a pseudorandom bit sequence: $b_1, b_2, b_3, \dots$


- The security of BBS depends on the **difficulty of factoring** the modulus $M$.  
- Without knowledge of $p$ and $q$, predicting the next bit from previous ones is computationally infeasible.  
- BBS is considered a **cryptographically secure PRNG (CSPRNG)**.



Blum Blum Shub (BBS) implementation in Python:

```python
def is_prime(n):
    if n < 2: return False
    for i in range(2, int(n**0.5)+1):
        if n % i == 0: return False
    return True

def next_prime(start):
    n = start
    while True:
        if is_prime(n): return n
        n += 1

def bbs_generate(bits, p=None, q=None, seed=None):
    # Choose primes p, q congruent to 3 mod 4
    if p is None: p = next_prime(383)  # small example primes
    if q is None: q = next_prime(503)
    if p % 4 != 3: p = next_prime(p)
    if q % 4 != 3: q = next_prime(q)
    
    M = p * q
    if seed is None: seed = 2
    x = seed**2 % M
    
    result = []
    for _ in range(bits):
        x = x**2 % M
        result.append(x % 2)  # LSB as output
    return result

# Generate 32 random bits
random_bits = bbs_generate(32)
print(random_bits)
```

---

# 2. Fundamentals of Stream Ciphers

A **stream cipher** is a type of symmetric encryption algorithm that encrypts data one bit or byte at a time, rather than processing fixed-size blocks (as in block ciphers). Stream ciphers are often used when data arrives continuously (e.g., network streams, voice communications).

A stream cipher generates a **keystream** — a sequence of pseudorandom bits derived from a secret key (and often an initialization vector, IV). Encryption and decryption are performed by combining the plaintext/ciphertext with the keystream using bitwise XOR:

**Encryption:**  

$$C_i = P_i \oplus K_i$$

**Decryption:**  

$$P_i = C_i \oplus K_i$$

where:  
- $P_i$ = plaintext bit/byte,  
- $C_i$ = ciphertext bit/byte,  
- $K_i$ = keystream bit/byte.

Stream ciphers are fast and efficient for real-time applications. Keystream must never repeat with the same key (reusing keystreams is insecure). Security depends on the unpredictability of the keystream.

We will consider **RC4** (historically popular, now considered insecure) and **Salsa20/ChaCha** (modern, widely used in TLS, VPNs, etc.).

---

# 3. RC4 


## Key Scheduling Algorithm 

RC4 uses a permutation array $S$ of size $N = 256$ and two indices $i, j$. The key $K$ has length $L$, with $1 \le L \le 256$.

The algorithm:

```
for i from 0 to 255
    S[i] := i
endfor
j := 0
for i from 0 to 255
    j := (j + S[i] + key[i mod keylength]) mod 256
    swap values of S[i] and S[j]
endfor
```

## Pseudo-Random Generation Algorithm (PRGA)

Each keystream byte $z$ is generated by:

```
i := 0
j := 0
while GeneratingOutput:
    i := (i + 1) mod 256
    j := (j + S[i]) mod 256
    swap values of S[i] and S[j]
    t := (S[i] + S[j]) mod 256
    K := S[t]
    output K
endwhile
```

This produces a stream of K[0], K[1], ... which are XORed with the plaintext to obtain the ciphertext. So 

$$ciphertext[l] = plaintext[l] ⊕ K[l].$$


### Security


#### Two structural weak points 
1. **Weak key-scheduling (KSA) correlations.** The KSA does not sufficiently mix key material into the internal permutation, so internal state bytes and early keystream bytes remain biased and leak information about the key.  
2. **Stream-cipher malleability.** Like all stream ciphers, RC4 is trivially malleable: flipping a keystream bit flips the corresponding plaintext/ciphertext bit. Without an authenticating MAC, this allows bit-flipping and forgery-style attacks.

These two facts together produce both practical passive attacks (recover plaintext / keys from many encryptions) and active attacks (inject or modify ciphertexts).



#### Related-key / nonce-handling pitfalls
RC4 **does not accept a separate nonce** in its original design. Protocols that must use a long-term key for many encryptions therefore need a secure way to derive per-message/ session keys. Two common patterns:

- **Good:** derive the per-session RC4 key as a cryptographic hash (or KDF) of the long-term key and nonce (so the RC4 key material is unrelated between sessions).  
- **Bad (widely seen historically):** simply *concatenate* nonce || key (or key || nonce). Because the KSA is weak, related-key effects appear — the attacker can collect many sessions whose keys differ only in the nonce prefix/suffix and exploit biases to recover the long-term key (this is the core idea behind the Fluhrer–Mantin–Shamir family of attacks and the practical WEP break).

**Rule:** *never* produce RC4 keys for multiple messages by naïve concatenation of nonce and long-term key.



#### Famous biases and what they mean 

RC4 keystream bytes are not uniformly random. A few illustrative facts:

- [Fluhrer, Mantin and Shamir showed](https://www.mattblaze.org/papers/others/rc4_ksaproc.pdf) the second keystream byte is biased toward zero with probability $1/128$ rather than the uniform $1/256$.  
- More generally, many early outputs and internal permutation positions show statistical biases (Roos, Paul & Maitra, Klein, and others analyzed and proved many such correlations).

A simple way to express the observable difference: if the true probabilities were $p_{\text{biased}}=1/128$ and $p_{\text{uniform}}=1/256$, the **excess probability** per sample is

$$
\Delta p \;=\; \frac{1}{128}-\frac{1}{256}\;=\;\frac{1}{256}.
$$

Over $N$ independent observations the expected excess count of that symbol is $N\Delta p = \dfrac{N}{256}$. For example, with $N=256$ samples the expected excess is $1$ — giving a very small but measurable signal that modern statistical techniques can amplify given many connections/observations.

**Consequence:** attackers with large numbers of ciphertexts (or many TLS handshakes, or many WEP frames) can distinguish RC4 outputs from random and ultimately recover plaintext or key material.



#### Protocol history and final status
RC4 was once widely used in SSL/TLS and wireless protocols (WEP/WPA-TKIP). The Fluhrer–Mantin–Shamir work was a practical cause of WEP's collapse and the push to WPA/WPA2.  

Although RC4 avoided some block-cipher-specific CBC attacks (e.g., BEAST), successive RC4-specific attacks (Royal Holloway, NOMORE, Bar Mitzvah, etc.) made RC4 untenable in TLS. 

**RFC 7465 (Feb 2015)**. RC4 is prohibited for use in TLS. In practice, major browsers and servers removed or disabled RC4 support.

**Bottom line:** RC4 is deprecated and should not be used in new designs.

#### Practical recommendations 
1. **Do not use RC4.** Prefer modern, authenticated ciphers (e.g., AEAD primitives such as AES-GCM, ChaCha20-Poly1305).  
2. If you must analyze legacy RC4 traffic for research/forensics, be aware of the specific biases (Mantin–Shamir, Roos, Klein, PTW, Royal Holloway improvements) and that large volumes of ciphertext greatly simplify attacks.  
3. **Never** derive per-session RC4 keys by simple concatenation of nonce and master key. Use a secure KDF/HMAC to derive independent keys.  
4. **Always authenticate.** Pair any stream cipher with a strong MAC or use an AEAD construction to avoid malleability.  
5. If maintaining legacy systems, push to migration plans: RC4 removal from TLS/WPA stacks is widely available and recommended.

---

# 4. Salsa20

[Salsa20 is a fast stream cipher by Daniel J. Bernstein](https://cr.yp.to/snuffle/spec.pdf). It generates a pseudorandom keystream which you XOR with plaintext to encrypt. It uses a 4×4 matrix of 32-bit words (512 bits) built from: constants, the key (128 or 256 bits), a nonce, and a block counter.


**How it mixes.**  
Mixing uses only three operations: 32-bit addition (mod $2^{32}$), XOR, and fixed-amount left-rotations (an ARX design). After 20 rounds (the usual Salsa20/20), the output block is the wordwise sum of the post-round state and the initial state:

$$
K_i = \bigl(X_i^{\text{final}} + X_i^{\text{initial}}\bigr)\bmod 2^{32},\quad i=0\dots15
$$

These 16 words form a 512-bit keystream block.

**Nonce and counter.**  
Use a unique nonce per message and a counter for block position. There’s also XSalsa20 for a longer nonce (192 bits) when needed.

**Security.**  
No practical attack breaks the full 20-round Salsa20; only reduced-round versions have theoretical cryptanalysis. Salsa20/12 and Salsa20/20 are considered safe in practice.

**Performance & implementation.**  
Very fast in software because it uses simple CPU-friendly ops and avoids table lookups (helps against timing attacks). Works well on x86, ARM, and embedded CPUs. Implementations are straightforward and available in libs like NaCl/libsodium.

**Comparison.**  
- **ChaCha20:** a close variant with different rotations and slightly faster diffusion; often preferred today for SIMD and standardization.  
- **AES (CTR):** AES can be faster *with* hardware AES-NI, but without hardware support Salsa20 often outperforms AES in software. Salsa20 is simpler to implement in constant time.

**Typical uses.**  
Secure software encryption (NaCl/libsodium), network protocols (e.g., DNSCurve/DNSCrypt historically), and as a building block in other primitives (e.g., scrypt).

**Bottom line.**  
Salsa20 = simple ARX design, very fast in software, safe when using the full 20 rounds and unique nonces.


# 5. Strumok — Ukrainian Stream Cipher (DSTU 8845:2019)

Strumok is a word-oriented symmetric stream cipher, standardized in Ukraine (DSTU 8845:2019).

### Basic Parameters

- Works with **64-bit words**. 
- Key lengths: **256 bits** or **512 bits**. 
- Initialization Vector (IV): 256 bits, public.
- Internal state size: composed of a Linear Feedback Shift Register (LFSR) of length 16 over the field $F_{2^{64}}$, plus a finite state machine with two 64-bit registers. 

**More details in**  [paper.](https://nure.ua/wp-content/uploads/2018/Scientific_editions/rvmnts_2018_193_4.pdf)

### Security remarks

Strumok adapts ideas from SNOW 2.0 but changes word size, LFSR polynomial, FSM details, and initialization.  

Some analyses highlight non-ideal aspects in the $512$-bit key mode: the IV usage may be partially redundant and certain guess-and-determine attacks reduce the effective complexity compared to naive brute force.  

Standard design goals: long period, high throughput on $64$-bit platforms, and resistance to known stream-cipher attacks (algebraic, correlation, guess-and-determine, slide attacks).
