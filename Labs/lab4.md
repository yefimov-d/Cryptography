# Assignment 4. Stream Ciphers

For some tasks you will need your email. Perform the transformation as in the example:

```
r.gosling_fit_13_23_b_d@knute.edu.ua -> rgosling
```
That is:
- take the part of the email **before `@`**,  
- remove fragments like `_fit_...`, `_b_d`, etc.,  
- remove all underscores `_`,  
- **the result is the concatenation of the initial and the surname.**

---

## Question 1 — RC4

Encrypt the cleaned email using [RC4](https://pycryptodome.readthedocs.io/en/latest/src/cipher/arc4.html) with the key ``0123456789ABCDEF0123456789ABCDEF``.

Provide the resulting ciphertext in **hexadecimal form**.

---

## Question 2 — Salsa20

Encrypt the cleaned email using [Salsa20](https://pycryptodome.readthedocs.io/en/latest/src/cipher/salsa20.html) with the key 
``A1B2C3D4E5F60718293A4B5C6D7E8F901112131415161718192A2B2C2D2E2F30``.

Use an 8-byte sequence of all zero bytes as a nonce.
Provide the resulting ciphertext in **hexadecimal form**.

---

## Optional task: Implement STRUMOK

Implement the **STRUMOK** stream cipher. Your implementation should be able to encrypt and decrypt text using a given key.

**Requirements:**

1. Write a function `encrypt(text, key)` that takes a plaintext string and a key, and returns the ciphertext.
2. Write a function `decrypt(ciphertext, key)` that takes a ciphertext string and the same key, and returns the original plaintext.
3. Ensure that your implementation correctly handles arbitrary-length input.
4. Include a simple demonstration showing encryption and decryption of a sample text.
5. Add comments explaining the logic of your implementation.

**References**  

Paper: https://periodicals.karazin.ua/cscs/article/view/12017