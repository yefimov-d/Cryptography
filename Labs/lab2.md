# Assignment 2. Historical Overview of Cryptographic Methods for Information Protection

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

## Question 1 — Caesar Cipher  
Encrypt your transformed email username using the **Caesar cipher**.  
**Key**: the first letter of your transformed email.  


---

## Question 2 — Vigenère Cipher  
Encrypt the same transformed username using the **Vigenère cipher**.  
**Key**: the first three letters of your transformed email.  


---

## Question 3 — Binary XOR of Username Letters  
Take the transformed email and:  
1. Convert each character into its **8-bit ASCII binary** form.  
2. Group the first **2 letters** (16 bits).  
3. Group the next **2 letters** (16 bits).  
4. XOR these two 16-bit blocks together. 

The result is 16 bits (presented in binary). For example, `0110001011101100`.

---

## Question 4 — One-Time-Pad

Let the Ukrainian alphabet be indexed $0,\dots,N-1$ (take $N=33$). Using additive one-time-pad encryption  

$$
C_i = (P_i + K_i) \bmod N,
$$

determine the key which, when applied to the plaintext **«муха»**, produces the ciphertext **«слон»**. Show your letter-to-index mapping and all intermediate calculations.

The answer must be given as a sequence of Ukrainian lowercase letters without quotation marks.

---

## Question 5 — Polybius square

Encrypt the **transformed email** using the Polybius square (English alphabet). Apply **Method 1** from the lecture to obtain the ciphertext.

The answer must be given as a sequence of English lowercase letters without quotation marks.

---

## Question 6 — Cardano Grille

Consider the following 4×4 Cardano grille key (holes marked with `O`):

|     | **1** | **2** | **3** | **4** |
|-----|-------|-------|-------|-------|
| **1** |     |       |       |   O    |
| **2** |      |       |    O   |      |
| **3** |   O   |      |       |  O     |
| **4** |       |       |     |       |

Use this grille to encrypt the first 16 letters of your transformed email. Rotate the encryption grille clockwise. 
If the email has fewer than 16 letters, pad the remaining positions with `a`.  

Provide the ciphertext as a sequence of English lowercase letters, without quotation marks.