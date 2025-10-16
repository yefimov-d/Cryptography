# Assignment 6. Key Exchange

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

## Question 1 — The Discrete Logarithm Problem

Let $p = 257$ and $g = 3$.  
Let $y$ be the **decimal ASCII code** of the **last letter** of your cleaned email.

Find the integer $x$ such that  

$$
g^x \equiv y \pmod{p}.
$$

---

## Question 2 — Diffie–Hellman Key Exchange (2048-bit MODP Group)

Alice and Bob would like to use the Diffie–Hellman algorithm to generate a shared secret for symmetric encryption. They use the [2048-bit MODP Group](https://www.rfc-editor.org/rfc/rfc3526.html?utm_source=chatgpt.com#page-3) from the [Internet Key Exchange (IKE) protocol](https://en.wikipedia.org/wiki/Internet_Key_Exchange).

The generator is: $g = 2$.
The prime is defined as:

$$
p = 2^{2048} - 2^{1984} - 1 + 2^{64} \cdot \left( \lfloor 2^{1918} \pi \rfloor + 124476 \right)
$$

Its hexadecimal value is:

```
FFFFFFFF FFFFFFFF C90FDAA2 2168C234 C4C6628B 80DC1CD1
29024E08 8A67CC74 020BBEA6 3B139B22 514A0879 8E3404DD
EF9519B3 CD3A431B 302B0A6D F25F1437 4FE1356D 6D51C245
E485B576 625E7EC6 F44C42E9 A637ED6B 0BFF5CB6 F406B7ED
EE386BFB 5A899FA5 AE9F2411 7C4B1FE6 49286651 ECE45B3D
C2007CB8 A163BF05 98DA4836 1C55D39A 69163FA8 FD24CF5F
83655D23 DCA3AD96 1C62F356 208552BB 9ED52907 7096966D
670C354E 4ABC9804 F1746C08 CA18217C 32905E46 2E36CE3B
E39E772C 180E8603 9B2783A2 EC07A28F B5C55DF0 6F4C52C9
DE2BCBF6 95581718 3995497C EA956AE5 15D22618 98FA0510
15728E5A 8AACAA68 FFFFFFFF FFFFFFFF
```

Define Alice’s private key as the **ASCII value of the first letter** of the cleaned email (lowercase, without symbols).  Define Bob’s private key as the **ASCII value of the second letter** of the cleaned email.  

Compute Alice’s public value, Bob’s public value, shared secret.

---

## Question 3 — Active attack on Diffie–Hellman

You are the attacker in the middle. You are given the public parameters `p,g`, the public values `A` and `B`, and the ciphertext (hex) that Alice sends to Bob (ciphertext and any metadata needed for decryption will be provided). **Your goal is to recover the plaintext.**

1. Recover the shared secret `S` (integer) used by Alice and Bob.  
2. Derive the 256-bit AES key `K` using [PBKDF2-HMAC-SHA256](https://docs.python.org/3/library/hashlib.html) with the exact parameters below.
3. Decrypt the provided ciphertext with AES-256-ECB and PKCS#7 padding using key `K`.


**Public parameters** 
Prime modulus: `p = 257` , generator: `g = 3`.  

**Public values** 
Alice’s public value: `A = 201`
Bob’s public value:   `B = 45`.   

**Ciphertext**
```febe49ef11b07faaec4a1c77cc5ab5f1bd8c4967d68092e6bd6ea8f9e928ef6f```

**Encryption scheme:**
Shared secret is used to derive an AES key via PBKDF2:
  - `K = pbkdf2_hmac("sha256", decimal value of shared secret, salt=b'\x00' * 16, iterations=200000, dklen=32)`

Symmetric encryption: **AES-256 in ECB mode** with **PKCS#7** padding.

