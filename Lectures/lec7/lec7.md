# Topic 7. Asymmetric Cryptosystems

## Lecture Plan

1. Symmetric Cryptosystem vs Asymmetric Cryptosystems. The three-pass protocol

2. Pseudoprime numbers.

3. RSA Cryptosystem

4. ElGamal Cryptosystem

---

## 1. Symmetric Cryptosystems vs Asymmetric Cryptosystems

Let's recall that cryptography uses two main types of systems to protect information: symmetric and asymmetric.

Often, modern systems use hybrid cryptography, where asymmetric algorithms securely exchange a session key, and symmetric algorithms then handle fast data encryption (as in HTTPS/TLS).

| Feature                           | Symmetric Cryptosystem                                                            | Asymmetric Cryptosystem                                                             |
| --------------------------------- | --------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **Key usage**                     | One secret/shared key used for both encryption and decryption.                    | Two keys: a **public key** (for encryption) and a **private key** (for decryption). |
| **Speed / performance**           | Faster; more efficient especially for large amounts of data.                      | Slower; more computationally intensive.                                             |
| **Key size for security**         | Smaller key sizes (e.g. 128-bit, 256-bit).                                        | Much larger key sizes (e.g. 2048-bit, 4096-bit).                                    |
| **Security properties**           | Provides confidentiality; needs additional mechanisms for integrity/authenticity. | Can provide confidentiality, authenticity, and non-repudiation.                     |
| **Key distribution / management** | Key must be securely shared between parties; difficult to scale.                  | Public key can be shared openly; private key stays secret.                          |
| **Typical use-cases**             | Bulk data encryption (files, VPN, storage, etc.).                                 | Digital signatures, key exchange, authentication, SSL/TLS.                          |
| **Overhead / resource usage**     | Low computational and memory overhead.                                            | High computational cost; slower for large data.                                     |
| **Resilience to key compromise**  | If the key is leaked, all data using that key is compromised.                     | Only private key must remain secret; public key compromise is harmless.             |
| **Examples**                      | AES, DES, 3DES, Blowfish, ChaCha20                                                | RSA, DSA, Diffie-Hellman, ECC (Elliptic Curve Cryptography)                         |


**Problem.** In symmetric cryptography both parties must share the same secret key $K$ for encryption/decryption. The central difficulty is **secure key distribution**: sending $K$ over an insecure channel lets an eavesdropper obtain it, breaking confidentiality.

#### The three-pass protocol
Instead of sending a shared key, each party keeps their own secret ($K_A$ for Alice, $K_B$ for Bob) and they exchange the *message* in three moves so that neither key is ever transmitted.

Let $E_K(\cdot)$ be encryption with key $K$ and $D_K(\cdot)$ the corresponding decryption. The protocol:

1. Alice to Bob: $C_1 = E_{K_A}(M)$  
2. Bob to Alice: $C_2 = E_{K_B}(C_1) = E_{K_B}(E_{K_A}(M))$  
3. Alice to Bob: $C_3 = D_{K_A}(C_2) = E_{K_B}(M)$  
4. Bob computes $M = D_{K_B}(C_3)$

After step 3 Bob removes his layer and recovers $M$. Neither $K_A$ nor $K_B$ was sent over the channel.

A crucial algebraic property needed is either **commutativity** of encryption or the ability to remove one layer without disturbing the other, i.e.

$$
D_{K_A}\big(E_{K_B}(E_{K_A}(M))\big)=E_{K_B}(M).
$$


**Advantages (why it matters)**
- No single shared secret is transmitted.  
- Each party retains their private key; an eavesdropper observing all messages cannot trivially recover $M$ without at least one private key.

**Limitations and practical remarks**
- **Commutative symmetric ciphers are rare.** Many practical symmetric schemes do not satisfy the required property, which limits direct application.  
- **No protection against active attacks.** A man-in-the-middle can substitute messages unless parties authenticate each other (e.g., with MACs or signatures).  
- **Public-key methods** (e.g., Diffie–Hellman, RSA) are usually preferred in practice for secure key exchange because they avoid the structural constraints and provide well-studied security properties.

The three-pass protocol is an elegant theoretical solution to the symmetric key distribution problem: it delivers a message without ever transmitting the shared key. Its real-world use requires special cryptographic properties and additional authentication to be secure in adversarial networks.

**Shamir’s protocol** is a concrete realisation of the three-pass (no-key) idea.

**Core instantiation (modular exponentiation).** Fix a prime $p$ with $M\in\mathbb{Z}_p^\times$.  
Alice picks secret $a$ such that $\gcd(a,p-1)=1$ and lets $a^{-1}$ be the inverse of $a$ modulo $p-1$.  
Bob picks secret $b$ with $\gcd(b,p-1)=1$ and inverse $b^{-1}\pmod{p-1}$.  
Encryption by exponentiation: $E_a(X)=X^a\pmod p$, $D_{a}(Y)=Y^{a^{-1}}\pmod p$.

**Protocol steps.**
1. Alice to Bob: $C_1 = E_a(M)=M^a\pmod p$.  
2. Bob to Alice: $C_2 = E_b(C_1)=M^{ab}\pmod p$.  
3. Alice to Bob: $C_3 = D_a(C_2)=\big(M^{ab}\big)^{a^{-1}}=M^b\pmod p$.  
4. Bob recovers $M = D_b(C_3)=\big(M^b\big)^{b^{-1}}\pmod p$.

The algebraic property used is that exponentiation composes and inverses exist modulo $p-1$:
$$D_a\big(E_b(E_a(M))\big)=E_b(M).$$

Shamir’s scheme is a no-key method using commutative operations (exponentiation). It’s instructive conceptually, but in practice requires authentication and relies on assumptions (e.g., discrete log hardness); modern deployments typically favour authenticated Diffie–Hellman or public-key primitives.

---

## 2. Pseudoprime numbers

#### Pseudoprime numbers

In number theory, **pseudoprime numbers** are composite numbers that, under certain tests, behave like prime numbers. They are not truly prime, but they can "deceive" some primality tests.

**Primality testing** is the process of determining whether a given number $n$ is **prime** or **composite**. Since prime numbers play a crucial role in cryptography and number theory, efficient methods for testing primality are essential.

A prime number is divisible only by $1$ and itself. A naïve approach to test primality is to check whether $n$ is divisible by any integer up to $\sqrt{n}$. While correct, this method becomes too slow for large numbers used in practice (such as in cryptographic applications).

#### Types of Primality Tests

1. **Deterministic Tests** . These tests always give the correct answer.
    - **Trial Division**: Check divisibility up to $\sqrt{n}$.  
    - **AKS Algorithm**: A polynomial-time algorithm that can determine primality with certainty.  
    - **Elliptic Curve Methods**: More advanced, used for large numbers.

2. **Probabilistic Tests**. These tests are much faster but give results with a small probability of error.  
   - **Fermat Test**: Based on Fermat’s Little Theorem. A composite number that passes the test for base $a$ is a **pseudoprime**.  
   - **Miller–Rabin Test**: An improved version of Fermat’s test, widely used in practice because it reduces the chance of error significantly.  
   - **Solovay–Strassen Test**: Uses properties of the Jacobi symbol and modular arithmetic.


For **small numbers**, deterministic tests are efficient enough. For **large numbers** (hundreds or thousands of digits, as in RSA encryption), **probabilistic tests** like Miller–Rabin are preferred because they are very fast and provide high confidence in primality.  
In cryptography, it is common to use several rounds of probabilistic testing with different bases to minimize the error probability.


#### Fermat Test Example

For a number $n$, choose a base $a$ such that $1 < a < n-1$ and $\gcd(a, n) = 1$.  
If $n$ is prime, **Fermat's Little Theorem** says that:

$$
a^{n-1} \equiv 1 \pmod{n}
$$

If this congruence fails, $n$ is composite.  A composite number $n$ that satisfies this congruence for a given base $a$ (where $\gcd(a, n) = 1$) is called a **pseudoprime to the base $a$**.


##### Example
The number $341 = 11 \times 31$ is not prime. However, for $a = 2$, we have:

$$
2^{340} \equiv 1 \pmod{341}
$$

Thus, $341$ is a **pseudoprime to base 2**.


##### Notes

- All prime numbers satisfy Fermat’s Little Theorem, but some composite numbers do as well — these are pseudoprimes.  
- A stronger category is called **Carmichael numbers**, which are composite numbers that behave like primes for **all integer bases** $a$ with $\gcd(a, n) = 1$.
- There are infinitely many pseudoprimes.  
- Pseudoprimes are **base-dependent**: a number may be pseudoprime for one base but not for another.  


#### Miller–Rabin Test Example

Given an odd integer $n > 2$, the test works as follows:

1. **Write $n-1$ in the form:**
   $$
   n - 1 = 2^s \cdot d
   $$
   where $d$ is odd, and $s \geq 1$.

2. **Choose a random base $a$** such that $1 < a < n - 1$.

3. **Compute:**
   $$
   x = a^d \bmod n
   $$

4. **Check conditions:**
   If $x = 1$ or $x = n - 1$, then $n$ passes this round of the test (it is a "probable prime").  
   Otherwise, repeat up to $s-1$ times:
     $$
     x \leftarrow x^2 \bmod n
     $$
     If $x = n - 1$, then $n$ passes this round (probable prime).  
     If $x = 1$ before reaching $n - 1$, then $n$ is **composite**.

5. **Decision:**
   If $n$ fails for a given base $a$, then $n$ is definitely **composite**.  
   If $n$ passes for several randomly chosen bases, then $n$ is **probably prime**.


If $n$ is composite, the probability that it passes the test for a random base $a$ is at most $\tfrac{1}{4}$.  
Repeating the test with multiple random bases reduces the error probability exponentially.  
If $n$ passes the test for many different bases, we accept $n$ as a **probable prime** with high confidence.

Example. **Input:** `n = 13`, `k = 2`

Step 1: Express $ n - 1 $ as $ d \cdot 2^r $. We need integers $ d $ and $ r $ such that:

$$
13 - 1 = 12 = 3 \times 2^2
$$

So, `d = 3`, `r = 2`.


Step 2: Repeat the test `k = 2` times



**First Iteration**

Pick a random number `a` in the range `[2, n - 2] = [2, 11]`.  Suppose `a = 4`.

Compute  

$$
x = a^d \bmod n = 4^3 \bmod 13 = 64 \bmod 13 = 12
$$

Since `x = n - 1 = 12`, this iteration **passes** (n is probably prime for this test).


**Second Iteration**

Pick another random number `a` in `[2, 11]`.  Suppose `a = 5`.

Compute  

$$
x = a^d \bmod n = 5^3 \bmod 13 = 125 \bmod 13 = 8
$$

`x` is neither `1` nor `12`, so continue with the next step.

Repeat the squaring step `(r - 1) = 1` time:

$$
x = x^2 \bmod n = 8^2 \bmod 13 = 64 \bmod 13 = 12
$$

Since `x = n - 1 = 12`, this iteration **passes**.


**Result**

Both iterations returned **true**,  
so the algorithm concludes that **13 is probably prime**.



#### Algorithms for Generating Prime Numbers

Prime numbers are essential in cryptography and many areas of computer science. Since large primes are needed in practice (for example, in RSA encryption), efficient **algorithms for generating prime numbers** have been developed.

##### 1. Random Generation with Primality Testing
One of the most common methods is:
1. Generate a random large odd number $n$ of the desired bit length.  
2. Test $n$ for primality using a **probabilistic primality test** (e.g., Miller–Rabin).  
3. If $n$ passes the test, accept it as a prime candidate; otherwise, generate another number and repeat.  

This method is practical because probabilistic tests are very fast, and the density of primes among large integers is high.


##### 2. Sieve of Eratosthenes
For smaller ranges of numbers, the **Sieve of Eratosthenes** is efficient:
- Start with a list of integers from $2$ to $N$.  
- Repeatedly mark multiples of each prime number as composite.  
- The remaining unmarked numbers are primes.  

Example: For $N = 120$, the sieve produces primes 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101, 103, 107, 109, 113.

![1](https://upload.wikimedia.org/wikipedia/commons/9/94/Animation_Sieve_of_Eratosth.gif)


##### 3. Other methods
- Sieve of Atkin (an advanced sieve algorithm that is more efficient than Eratosthenes for very large ranges). 

- Deterministic Primality-Based Construction (for example, **AKS primality test**).

#### UUIDv4

A [UUID](https://dou.ua/forums/topic/52305/) (Universally Unique Identifier) version 4 is a 128-bit identifier used to uniquely label objects or data. UUIDv4 uses random or pseudo-random numbers. The specific pseudo-random generator employed for UUIDv4 generation varies by implementation. UUIDv4 is **not an encryption tool** and does not provide confidentiality or integrity by itself. It only ensures a very low probability of collisions.

**Some use cases**: 
- **Database Keys**. As unique primary keys in distributed databases to avoid collisions without central coordination.  
- **Session IDs**. For identifying user sessions in web applications securely and uniquely.  
- **API Keys / Tokens**. To generate non-predictable identifiers for authentication or access control.  

---

# 3. RSA Cryptosystem

The **RSA cryptosystem** is one of the most widely used public-key cryptographic algorithms. It was introduced in 1977 by **Rivest, Shamir, and Adleman**, whose initials give the name RSA. Its security is based on the mathematical difficulty of **factoring large composite numbers**.

**Protocols Using RSA**

TLS/SSL (HTTPS) – RSA used for key exchange and authentication.

PGP (Pretty Good Privacy) – RSA encrypts session keys and creates digital signatures.

SSH (Secure Shell) – Supports RSA keys for authentication.

S/MIME (Secure Email) – Uses RSA for signing and encrypting email.

RSA is commonly used to sign (see the next lecture) documents, software, and certificates, ensuring authenticity and integrity. For example, code signing for software distribution often relies on RSA signatures.

RSA is the core algorithm in the X.509 certificates that underpin the TLS/SSL protocol, securing web traffic (HTTPS). Most Certificate Authorities (CAs) issue certificates signed with RSA keys.

#### Textbook RSA

In the **textbook version** of RSA:

1. **Key Generation**  
   - Choose two large prime numbers $p$ and $q$.  
   - Compute $n = p \cdot q$.  
   - Compute Euler’s totient: $\varphi(n) = (p-1)(q-1)$.  
   - Choose an integer $e$ such that $1 < e < \varphi(n)$ and $\gcd(e, \varphi(n)) = 1$.  
   - Compute $d$ as the modular inverse of $e$ modulo $\varphi(n)$:

$$
e \cdot d \equiv 1 \pmod{\varphi(n)}
$$

   - Public key: $(e, n)$  
   - Private key: $(d, n)$

2. **Encryption**  
   Given plaintext message $M$ (as an integer $0 < M < n$):

$$
C = M^e \bmod n
$$

3. **Decryption**  
   Recover message:

$$
M = C^d \bmod n
$$

#### Example

Key Generation

**Choose two distinct prime numbers**: 

$$
p = 61, \quad q = 53
$$

**Compute the modulus**:  

$$
n = p \cdot q = 61 \times 53 = 3233
$$

**Compute Euler's totient function**:  

$$
\varphi(n) = 60 \times 52 = 3120
$$

**Choose a public exponent $e$** such that $1 < e < \varphi(n)$ and $\gcd(e, \varphi(n)) = 1$:  

$$
e = 17
$$

It must satisfy:

$$
1 < e < \varphi(n), \quad \text{and} \quad \gcd(e, \varphi(n)) = 1
$$

Indeed, $\gcd(17, 3120) = 1$, so $e$ is valid.

**Compute the private exponent $d$** as the modular inverse of $e$ modulo $\varphi(n)$:  

$$
d = e^{-1} \bmod \varphi(n) = 2753
$$

Suppose we have a plaintext message:

$$
m = 65
$$

To encrypt, compute:

$$
c = m^e \bmod n = 65^{17} \bmod 3233  = 2790.
$$


So the **ciphertext** is:

$$
\boxed{c = 2790}
$$

**Decryption**

To decrypt, compute:

$$
m = c^d \bmod n = 2790^{2753} \bmod 3233 = 65.
$$

The decrypted message matches the original plaintext.



#### Why messages must be smaller than \(n\)

RSA works in the modular arithmetic system with modulus \(n = p \cdot q\). This means that the plaintext message, represented as an integer \(m\), must satisfy  
\[
0 < m < n
\]  
Otherwise, the encryption will not be valid.  

In practice, real RSA does not directly encrypt arbitrary messages. Instead, the message is first encoded and padded (for example with **PKCS#1 v1.5** or **OAEP**). Padding ensures both security (avoiding predictable ciphertexts) and correctness (splitting long texts into chunks smaller than \(n\)).


#### Longer text encryption: need for larger primes

If we want to encrypt more than one byte (e.g., "Hi"), then the integer representation of the message is already larger than 3233, which is not allowed in our toy example. To solve this, we need larger primes so that \(n\) becomes much bigger, or we must split the message into blocks that each fit into \(n\).



#### RSA-OAEP


Padding schemes randomize the plaintext before encryption to make the ciphertexts unique and secure.  
**PKCS#1 v1.5 padding** (older, less secure).  
**OAEP (Optimal Asymmetric Encryption Padding)** — a modern padding scheme providing semantic security.

![OAEP](https://upload.wikimedia.org/wikipedia/commons/8/8f/OAEP_encoding_schema.svg)

#### Notation and parameters

- `MGF` — mask generation function (commonly `MGF1`).  
- `Hash` — cryptographic hash function chosen for OAEP.  
- `hLen` — output length of `Hash` in bytes.  
- `k` — length in bytes of the RSA modulus $n$ (i.e. the octet-length of the modulus).  
- $M$ — the plaintext message to be padded; its length is $mLen$ bytes, with the constraint  

$$mLen \le k - 2\cdot hLen - 2.$$  

- $L$ — optional label associated with the message (by default $L$ is the empty string).  
- `PS` — padding string composed of null bytes (`0x00`), of length 

$$k - mLen - 2\cdot hLen - 2.$$

- `||` denotes concatenation.  
- $\oplus$ denotes bitwise XOR.



#### Encoding (OAEP)

To produce the encoded message `EM` of length $k$ bytes:

1. Compute the hash of the label:  
   $$\text{lHash} = \text{Hash}(L).$$

2. Build the padding string `PS` consisting of $$k - mLen - 2\cdot hLen - 2$$ bytes, each equal to `0x00`.

3. Form the data block `DB` by concatenating `lHash`, `PS`, a single byte `0x01`, and the message $M$:  
   $$\text{DB} = \text{lHash} \;||\; \text{PS} \;||\; \text{0x01} \;||\; M.$$  
   The length of `DB` is $k - hLen - 1$ bytes.

4. Generate a random `seed` of length $hLen$ bytes.

5. Compute a mask for `DB` using the mask generation function:  
   $$\text{dbMask} = \text{MGF}(\text{seed},\; k - hLen - 1).$$

6. Mask the data block:  
   $$\text{maskedDB} = \text{DB} \oplus \text{dbMask}.$$

7. Compute a mask for the `seed`:  
   $$\text{seedMask} = \text{MGF}(\text{maskedDB},\; hLen).$$

8. Mask the seed:  
   $$\text{maskedSeed} = \text{seed} \oplus \text{seedMask}.$$

9. Assemble the encoded message `EM` as the single leading byte `0x00` followed by `maskedSeed` and `maskedDB`:  
   $$\text{EM} = \text{0x00} \;||\; \text{maskedSeed} \;||\; \text{maskedDB}.$$

The resulting `EM` is then suitable for RSA encryption (i.e. treated as an integer and exponentiated modulo $n$).



##### Security of RSA

RSA security depends on the hardness of factoring the modulus  `n = p · q`. If an attacker can factor `n` they immediately recover the private key. For an accessible practical introduction, see the *Handbook of Applied Cryptography* (Chapter 8).  
[Handbook of Applied Cryptography — HAC (Chapter 8 (PDF)](https://cacr.uwaterloo.ca/hac/about/chap8.pdf).

**Textbook RSA** is deterministic (same message → same ciphertext) and malleable (ciphertext manipulations map to predictable plaintext changes). That enables chosen-ciphertext and many other attacks. 


In practice, RSA is always combined with **padding schemes** or used in conjunction with symmetric keys. Use OAEP for encryption and PSS for signatures (as recommended in modern PKCS#1) and ensure implementations do **not** leak padding/formatting errors. See PKCS#1 (RFC 8017) for the modern recommendations.  
[PKCS#1: RSA Cryptography Specifications (RFC 8017)](https://www.rfc-editor.org/rfc/rfc8017.html).

The choice of `Hash` and `MGF` should be fixed for a given RSA key pair (PKCS#1 recommends `MGF1` with an appropriate `Hash`). Use a cryptographically secure RNG for `seed`, and avoid side-channel leaks in checks and error messages.

[OpenSSL can be used to generate and examine a real keypair.](https://en.wikibooks.org/wiki/Cryptography/Generate_a_keypair_using_OpenSSL)


Where to read next:

- [Handbook of Applied Cryptography (HAC) — chapters on RSA & factoring](https://cacr.uwaterloo.ca/hac/)  
- [PKCS#1 (RFC 8017) — RSA specifications, OAEP & PSS](https://www.rfc-editor.org/rfc/rfc8017.html)  
- [Bleichenbacher, *Chosen Ciphertext Attacks Against Protocols Based on the RSA Encryption Standard PKCS #1* (1998).](https://archiv.infsec.ethz.ch/education/fs08/secsem/bleichenbacher98.pdf)  

- [Kocher, *Timing Attacks on Implementations of Diffie-Hellman, RSA, DSS, and Other Systems* (CRYPTO 1996).](https://paulkocher.com/doc/TimingAttacks.pdf)  
- [Boneh, Joux, Nguyen, *Why Textbook ElGamal and RSA are Insecure* (ASIACRYPT 2000).](https://crypto.stanford.edu/~dabo/pubs/abstracts/ElGamalattack.html)



---

## 4. ElGamal Cryptosystem

The ElGamal cryptosystem was proposed by Taher ElGamal in 1985 and is based on the computational hardness of the discrete logarithm problem. The system is used for encryption and for constructing digital signature schemes.

The ElGamal cryptosystem can be viewed as an application of the Diffie-Hellman key exchange for producing ephemeral shared keys. Note that ElGamal encryption is distinct from the ElGamal digital signature algorithm.

GNU Privacy Guard (GPG) – Implements ElGamal for encryption and key management.

Electronic Voting Protocols – ElGamal’s homomorphic property is used in tallying encrypted votes without decrypting individual ballots.

Mix-Nets and Anonymity Protocols – ElGamal re-encryption is used to shuffle and anonymize ciphertexts in systems providing strong privacy guarantees.

But RSA dominates in practice due to its integration into web security (TLS/SSL), digital signatures, and public key infrastructure (PKI). ElGamal is less widespread but important in research, privacy-preserving protocols, and as the basis for signature schemes (DSA, ECDSA).


##### Key Generation

1. A random prime number $p$ is generated.  
2. An integer $g$ is chosen as a primitive root modulo $p$.  
3. A random integer $x$ is chosen such that $1 < x < p - 1$.  
4. Compute $$y = g^x \bmod p.$$

The public key is the triple $(y, g, p)$ and the private key is the integer $x$.

##### Encryption

Let the plaintext message be $M$ with $M < p$. Encryption proceeds as follows:

1. Choose a random session key $k$ such that $1 < k < p - 1$.  
2. Compute

$$a = g^k \bmod p$$

and

$$b = y^k \, M \bmod p.$$

3. The ciphertext is the pair $(a, b)$.

Note that the ciphertext length in ElGamal is twice the length of the original message $M$.

##### Decryption

Knowing the private key $x$, recover the plaintext $M$ from ciphertext $(a,b)$ by

$$M = b \cdot (a^x)^{-1} \bmod p.$$

One easily verifies that

$$(a^x)^{-1} = g^{-kx} \bmod p$$

and therefore

$$b \cdot (a^x)^{-1} = (y^k M) \cdot g^{-xk} \equiv (g^{xk} M) \cdot g^{-xk} \equiv M \pmod{p}.$$

For practical computation one often uses the equivalent formula

$$M = b \cdot a^{(p-1-x)} \bmod p.$$

##### Example

**Encryption**

1. Suppose the plaintext is $M = 5$.  
2. Key generation:
   * Let $p = 11$, $g = 2$.  
   * Choose $x = 8$.  
   * Compute $$y = g^x \bmod p = 2^8 \bmod 11 = 3.$$  
   * Public key: $(p,g,y) = (11,2,3)$. Private key: $x = 8$.
3. Choose $k = 9$.  
4. Compute $$a = g^k \bmod p = 2^9 \bmod 11 = 6.$$  
5. Compute $$b = y^k \cdot M \bmod p = 3^9 \cdot 5 \bmod 11 = 9.$$  
6. Ciphertext: $(a,b) = (6,9)$.

**Decryption**

1. Given ciphertext $(a,b) = (6,9)$ and private key $x = 8$, compute
$$M = b \cdot (a^x)^{-1} \bmod p = 9 \cdot (6^8)^{-1} \bmod 11 = 5.$$
2. The recovered plaintext is $M = 5$.


Because the scheme uses a random value $k$, ElGamal is a probabilistic encryption scheme: the same plaintext $M$ encrypted with different random $k$ yields different ciphertexts. This randomness improves security but results in ciphertexts that are twice as long as the plaintext.

If the same $k$ is reused for two distinct messages $M$ and $M'$, producing ciphertexts $(a,b)$ and $(a',b')$ with the same $a$ value, then
$$b (b')^{-1} = M (M')^{-1},$$
which allows recovery of one message if the other is known. Therefore, in practice it is essential to use a fresh random $k$ for each encryption.

