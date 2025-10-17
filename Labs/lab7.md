# Assignment 7. Asymmetric Cryptosystems

For some tasks you will need your email. Perform the transformation as in the example:

```
m.jordan_fit_13_23_b_d@knute.edu.ua -> mjordan
```
That is:
- take the part of the email **before `@`**,  
- remove fragments like `_fit_...`, `_b_d`, etc.,  
- remove all underscores `_`,  
- **the result is the concatenation of the initial and the surname.**

---


## Question 1 — Textbook RSA

Use the following RSA parameters for all computations:

- Primes: $p = 11$, $q = 13$
- Public exponent: $e = 7$
- Plaintext message: $m = 9$

Compute the ciphertext
$$
c \equiv m^e \pmod{n}.
$$

Find the private exponent. Recover the original message using

$$
m \equiv c^d \pmod{n}.
$$

## Question 2 — RSA Key Generation with Pseudoprimes

This exercise demonstrates what goes wrong if the "primes" used are actually composite pseudoprimes. Suppose you have checked the numbers $p = 530881$ and $q = 552721$ with the Fermat test and use them as if they were primes (in fact, these are Carmichael numbers).

Compute:

- $n = p \cdot q$.
- The totient estimate $\varphi(n) = (p-1)(q-1)$.
- Choose the public exponent $e = 65537$ and verify that $\gcd(e, \varphi(n)) = 1$.
- Compute the private exponent $d$ such that $e \cdot d \equiv 1 \pmod{\varphi(n)}$.

Convert the byte string of your cleaned email [into an integer $m$ using big-endian byte order](https://stackoverflow.com/questions/50509017/how-is-int-from-bytes-calculated). Encrypt:

$$
c \equiv m^e \pmod n.
$$

Then attempt decryption with $d$:

$$
\hat m \equiv c^{d} \pmod n.
$$

Does $\hat m = m$? Report the result and explain numerically what happens.

## Question 3 — RSA-OAEP

Use the public exponent 65537 and [generate](https://pycryptodome.readthedocs.io/en/latest/src/public_key/public_key.html) your own RSA key pair with a key size of 2048 bits. Keep both the **public and private key**; the private key is required for verification.


**Encrypt your cleaned email** using [RSA-OAEP](https://pycryptodome.readthedocs.io/en/latest/src/cipher/oaep.html) with the following parameters:
   - Hash function **SHA-256**
   - [Mask Generation Function](https://en.wikipedia.org/wiki/Mask_generation_function) (MGF1) based on **SHA-256**
   - Label: empty (`b""`)

**Provide the following for verification**:
   - **Ciphertext** as a hexadecimal string.
   - **Your private key** (PEM format).
   - **Seed used in OAEP padding**.


## Question 4 — ElGamal Cryptosystem

Use the following ElGamal parameters for all computations:

- Prime modulus: $p = 17$
- Generator (primitive root modulo $p$): $g = 3$
- Private key: $x = 5$
- Plaintext message: $m = 7$
- Random ephemeral key: $k = 4$

Compute the public key component
$$
y \equiv g^x \pmod{p}.
$$

Compute the ciphertext pair $(c_1, c_2)$ using
$$
c_1 \equiv g^k \pmod{p}, \quad c_2 \equiv m \cdot y^k \pmod{p}.
$$

Recover the original message using
$$
m \equiv c_2 \cdot (c_1^x)^{-1} \pmod{p}.
$$

Verify that the decrypted message equals the original plaintext $m$.