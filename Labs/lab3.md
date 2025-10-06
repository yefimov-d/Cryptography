# Assignment 3. Block Ciphers

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

## Question 1 — DES
Encrypt the first block of your cleaned email using DES in ECB mode with the key: ``133457799BBCDFF1``

If the input data is **shorter than 8 characters**, pad it with `\x00` bytes (null bytes) to make it exactly 8 bytes. If the email is **longer**, only the first 8 characters are used.

Give the ciphertext in hex.

For example, you can use PyCryptodome library:
https://pycryptodome.readthedocs.io/en/latest/src/cipher/des.html

---

## Question 2 — 3DES
Encrypt the first block of your cleaned email using [3DES](https://pycryptodome.readthedocs.io/en/latest/src/cipher/des3.html) in ECB mode with keys:
``Key1 = 0123456789ABCDEF``  
``Key2 = 23456789ABCDEF01``  
``Key3 = 456789ABCDEF0123``

If the input data is **shorter than 8 characters**, pad it with `\x00` bytes (null bytes) to make it exactly 8 bytes. If the email is **longer**, only the first 8 characters are used.

Provide the ciphertext in hex.

---

## Question 3 — AES

Take the first 16 bytes of your cleaned email (pad with zeros if shorter). Encrypt it using [AES-128](https://pycryptodome.readthedocs.io/en/latest/src/cipher/aes.html) in ECB mode with the key:
``2b7e151628aed2a6abf7158809cf4f3c``

Provide the ciphertext in hex.

---

## Question 4 — ECB vs CBC Mode

1. Download [the image](https://www.osadl.org/fileadmin/dam/images/tux-72.png) and open with Pillow; convert to **RGB**.
2. Extract raw pixel bytes `P = img.tobytes()`; record `width×height` and `len(P)`.
3. Use pad function from ``Crypto.Util.Padding`` to pad `P` with **PKCS#7** to 16-byte boundary → `P_padded`.
4. Encrypt `P_padded` with AES-128:
   - Key = ASCII-derived 16-byte key from cleaned email, padded with zeros if shorter.
   - ECB → `C_ecb`
   - CBC → `C_cbc` (use IV = ```451652008A75BF26D4B86AEE5A2CDF81```)
   Save `C_ecb` and `C_cbc` as binary files.
5. For visualization: truncate each ciphertext to original `len(P)` and rebuild RGB images; save `tux_ecb.png`, `tux_cbc.png`.
6. Compute SHA-256 (hex) of:
   - `C_ecb` (full padded ciphertext)
   - `C_cbc` (full padded ciphertext)
   <!-- - `tux_ecb.png` file
   - `tux_cbc.png` file -->
7. Write a short note (≤300 words) explaining why ECB leaks structure and CBC does not.

Provide SHA-256 digests of ciphertexts as the answer.

---

## Optional task: Implement Kalyna

1. Implement Kalyna-128 (block=128 bits) with key=128 bits (rounds = 10).  
2. Implement encryption and decryption (full reversible algorithm).  
3. Include unit tests using official test vectors (ciphertext ⇄ plaintext).  
4. Provide a small CLI or script to encrypt/decrypt hex inputs and to run test vectors.

### Test vectors

**Kalyna-128/128 encryption**

``KEY:
 000102030405060708090A0B0C0D0E0F``

``PLAINTEXT:
 101112131415161718191A1B1C1D1E1F``

``CIPHERTEXT:
 81BF1C7D779BAC20E1C9EA39B4D2AD06``

**Kalyna-128/128 decryption**

``KEY:
 0F0E0D0C0B0A09080706050403020100``

``CIPHERTEXT:
 1F1E1D1C1B1A19181716151413121110``

``PLAINTEXT:
 7291EF2B470CC7846F09C2303973DAD7 ``



**References**

Paper: https://eprint.iacr.org/2015/650.pdf
Author's C implementation: https://github.com/Roman-Oliynykov/Kalyna-reference


