# Topic 2. Historical Overview of Cryptographic Methods for Information Protection

## Lecture Plan
1. History of Cryptography
2. Basic Concepts of Cryptography
3. The Role of Cryptography in Data Protection
4. Concept and Types of Ciphers
5. Cipher Requirements – Kerckhoffs's Principle
6. The Perfect Cipher and Classes of Cipher Strength

---

# 1. History of Cryptography

## 1.1 Ancient Encryption Methods

Before the rise of modern cryptography, ancient civilizations developed ingenious methods to conceal messages and protect secrets. These early techniques often relied on substitution, transposition, or physical artifacts that required special knowledge to decode. Although relatively simple by today’s standards, they laid the foundation for classical ciphers and modern cryptography.

### Scytale (Spartan Cipher)

![1](https://lockhouse.co.uk/wp-content/uploads/2020/09/Skytale-300x171.png)

One of the earliest known encryption devices was the **scytale**, used by the Spartans around 7th century BCE. It consisted of a wooden rod of a specific diameter around which a strip of parchment or leather was wound. The message was written across the wrapped strip, and when unwrapped, the letters appeared scrambled. Only someone with a rod of the same diameter could reconstruct the original text.

### Atbash Cipher
The **Atbash cipher** is a simple substitution cipher originating in ancient Hebrew texts. It replaces each letter of the alphabet with its counterpart from the opposite end. For example, A ↔ Z, B ↔ Y, C ↔ X, and so forth. While not particularly secure, Atbash was often used to conceal sacred or esoteric words.

If we assign each letter a number from **0 to 25** (A = 0, B = 1, …, Z = 25),  
the Atbash cipher can be expressed as:

$$
E(x) = (25 - x) \mod 26
$$

$$
D(x) = (25 - x) \mod 26
$$

where:
- $E(x)$ is the encryption function,  
- $D(x)$ is the decryption function (identical in Atbash),  
- $x$ is the numerical value of the plaintext letter.

Example:
- Plaintext: **A** → $x = 0$  

$$
E(0) = (25 - 0) \mod 26 = 25 \Rightarrow Z
$$

- Plaintext: **B** → $x = 1$  

$$
E(1) = (25 - 1) \mod 26 = 24 \Rightarrow Y
$$

- Plaintext: **C** → $x =2$  

$$
E(2) = (25 - 2) \mod 26 = 23 \Rightarrow X
$$

### Caesar Shift (Early Roman Substitution)
Julius Caesar employed a basic substitution method where each letter of the plaintext was shifted by a fixed number of positions in the alphabet (commonly three). For instance, A → D, B → E, C → F. This method, though simple, was effective in military communication at the time.

If we assign each letter a number from **0 to 25** (A = 0, B = 1, …, Z = 25),  
the Caesar cipher with shift $k$ can be expressed as:

**Encryption**

$$
E(x) = (x + k) \mod 26
$$

**Decryption**

$$
D(x) = (x - k) \mod 26
$$

where:
- $x$ is the numerical value of the plaintext letter,  
- $k$ is the fixed shift (commonly $k = 3$),  
- $E(x)$ is the encrypted value,  
- $D(x)$ is the decrypted value.


**Example (with $k = 3$):**
- Plaintext: **A** → $x = 0$  

$$
E(0) = (0 + 3) \mod 26 = 3 \Rightarrow D
$$

- Plaintext: **B** → $x = 1$

$$
E(1) = (1 + 3) \mod 26 = 4 \Rightarrow E
$$

- Plaintext: **X** → $x = 23$ 

$$
E(23) = (23 + 3) \mod 26 = 0 \Rightarrow A
$$


### Polybius Square

Invented by the Greek historian **Polybius** in the 2nd century BCE, the **Polybius square** mapped letters to pairs of numbers using a 5×5 grid. For example, A = 11, B = 12, C = 13, etc. It allowed for secure numeric communication, which could also be transmitted via torches, hand signals, or codes.

We assume the standard English alphabet with **I/J merged** (i.e., replace `J → I` before processing).

|     | 1 | 2 | 3 | 4 | 5 |
|-----|---|---|---|---|---|
| **1** | A | B | C | D | E |
| **2** | F | G | H | I/J | K |
| **3** | L | M | N | O | P |
| **4** | Q | R | S | T | U |
| **5** | V | W | X | Y | Z |

- Rows are numbered **1–5** (left side).  
- Columns are numbered **1–5** (top side).  
- Each letter is identified by its **row number** followed by its **column number**.

**Example: Encrypting "HELLO"**
1. **H** → row 2, column 3 → **23**  
2. **E** → row 1, column 5 → **15**  
3. **L** → row 3, column 1 → **31**  
4. **L** → row 3, column 1 → **31**  
5. **O** → row 3, column 4 → **34**

Coordinate representation: `23 15 31 31 34`

**Method 1 – Vertical Caesar (Shift by 5 / move one row down)**

**Rule:** Replace each letter with the one immediately **below** in the same column. Wrap from bottom to top.

| Plain | Location | Cipher |
|-------|----------|--------|
| H | (2,3) | N |
| E | (1,5) | K |
| L | (3,1) | Q |
| L | (3,1) | Q |
| O | (3,4) | T |

Ciphertext (Method 1):  NKQQT

**Method 2 – Bifid Cipher (No Key)**

Step 1: Coordinates of Plaintext
- H → (2,3)  
- E → (1,5)  
- L → (3,1)  
- L → (3,1)  
- O → (3,4)  

Step 2: Separate Rows and Columns
- Rows: `2 1 3 3 3`  
- Columns: `3 5 1 1 4`

Step 3: Concatenate Rows then Columns

2 1 3 3 3 3 5 1 1 4

Step 4: Regroup into Coordinate Pairs

(2,3) (1,3) (3,3) (3,5) (3,1)

Step 5: Convert Pairs Back to Letters Using Polybius Square
- (2,3) → H  
- (1,3) → C  
- (3,3) → N  
- (3,5) → P  
- (3,1) → L 

**Ciphertext (Method 2):**  HCNPL

- **Coordinates of "HELLO":** `23 15 31 31 34`  
- **Method 1 (Vertical Caesar):** `NKQQT`  
- **Method 2 (Bifid, no key):** `HCNPL`

### Steganography in Ancient Greece
Apart from ciphers, ancient cultures also practiced **steganography**—concealing the very existence of a message. Herodotus recounts how secret messages were hidden beneath wax on wooden tablets or tattooed on a servant’s shaved head, later revealed once the hair grew back.

- **Herodotus** wrote about these methods in his *Histories*, documenting various clever ways Greek states communicated secretly.  
- Steganography was seen as a valuable tool for spies and generals, allowing them to coordinate without alerting enemies.



## 1.2 Classical Ciphers  

### Homophonic Cipher

The **homophonic cipher** is a substitution cipher where a single plaintext letter can be replaced by multiple ciphertext symbols.  
- **Purpose:** Reduce frequency patterns to make cryptanalysis harder.  
- **Example:** Letter "E" (most frequent in English) could be represented by symbols like 12, 45, 78, etc.  
- **Characteristics:**  
  - Frequency distribution of ciphertext symbols is more uniform.  
  - Increases resistance against frequency analysis. 

- **Example Mapping:**  

| Plaintext | Cipher symbols   |
|-----------|------------------|
| E         | 12, 45           |
| H         | 31               |
| L         | 54, 63, 78       |
| O         | 87               |

- **Plaintext:** HELLO  
- **Ciphertext:** 31 12 54 78 87   

### Vigenère Cipher

The **Vigenère Cipher** is a method of encrypting alphabetic text using a **repeating keyword**. It is a type of **polyalphabetic substitution cipher**, which improves upon the simple Caesar cipher by using multiple shift values.

**How it Works**

1. **Choose a keyword**  
   Example: `KEY`

2. **Repeat the keyword** to match the length of the plaintext.  
   Example:  
   Plaintext:  `HELLO`  
   Keyword:    `KEYKE`

3. **Encryption formula**:  
   Each letter of the plaintext is shifted according to the corresponding letter of the keyword.  

$$
C_i = (P_i + K_i) \mod 26
$$  

   - $C_i$ = ciphertext letter  
   - $P_i$ = plaintext letter (A=0, B=1, … Z=25)  
   - $K_i$ = keyword letter (A=0, B=1, … Z=25)


**Example: Encrypting "HELLO" with keyword "KEY"**

| Plaintext | H | E | L | L | O |
|-----------|---|---|---|---|---|
| Keyword   | K | E | Y | K | E |
| Shift     | 10| 4 | 24| 10| 4 |
| Ciphertext| R | I | J | V | S |

**Result:** `RIJVS`


**Decryption**

To decrypt, reverse the shift using the keyword:

$$
P_i = (C_i - K_i + 26) \mod 26
$$

- Subtract the keyword letter value from the ciphertext letter value.
- Convert the numbers back to letters to get the plaintext.


**Features**

- Provides **better security than Caesar cipher**.
- Vulnerable to **Kasiski examination** if the keyword is short or repeated.
- Requires the sender and receiver to **share the keyword**.


**Notes**

- Keyword length affects security: longer, random keywords are more secure.
- Can be adapted to non-English alphabets by adjusting the modulo value.




### Cardano Grille Cipher

The **Cardano grille cipher** uses a physical grid (grille) with cut-out holes to write hidden messages.  
- **Mechanism:**  
  1. Place the grille over a blank sheet.  
  2. Write letters of the message in the holes.  
  3. Remove the grille and fill the remaining spaces with random letters.  
- **Characteristics:**  
  - Provides spatial concealment of the message.  
  - Can be rotated to hide multiple messages on the same sheet.  


<div style="display: flex; gap: 10px; align-items: center;">
  <img src="https://upload.wikimedia.org/wikipedia/commons/5/59/Tangiers2.svg" alt="Image 1" width="200" />
  <img src="https://upload.wikimedia.org/wikipedia/commons/9/97/Tangiers1.svg" alt="Image 2" width="200" />
</div>

### Playfair Cipher

The **Playfair cipher** is a digraph substitution cipher invented by Charles Wheatstone and popularized by Lord Playfair.  
- **Mechanism:**  
  - Uses a 5×5 square containing letters of the alphabet.  
  - Encrypts pairs of letters (digraphs) rather than single letters.  
  - Rules for encryption depend on the position of the letters in the square.  
- **Example:** Encrypt "HI" using a keyword square.  
- **Characteristics:**  
  - Encrypting digraphs reduces frequency analysis effectiveness.  
  - Simple to implement manually.


Step 1: Create the Playfair Square

1. Choose a **keyword** (e.g., `MONARCHY`).  
2. Fill a 5×5 square with the letters of the keyword, omitting duplicates.  
3. Fill the remaining empty spaces with the rest of the alphabet (usually merging I/J).  

**Example Square (Keyword: MONARCHY)**

| M | O | N | A | R |
|---|---|---|---|---|
| **C**| **H** | **Y** | B | D |
| E | F | G | I | K |
| L | P | Q | S | T |
| U | V | W | X | Z |

Step 2: Prepare the Plaintext

1. Break the message into **digraphs** (pairs of letters).  
2. If a pair has the **same letters**, insert a filler letter (usually `X`).  
3. If the message has an **odd number of letters**, add a filler at the end.

**Example:**  
Plaintext: `HELLO` → Digraphs: `HE LX LO`  

Step 3: Encrypt Each Digraph

Apply the following rules for each pair:

1. **Same row:**  
   - Replace each letter with the letter **immediately to its right** (wrap around to the start of the row if needed).  

2. **Same column:**  
   - Replace each letter with the letter **immediately below** (wrap around to the top if needed).  

3. **Rectangle (different row and column):**  
   - Replace each letter with the letter **in the same row but in the column of the other letter** (swap columns).  

**Example Digraph Encryption:**  

- Digraph: `HE`  
  - `H` is at row 2, column 2  
  - `E` is at row 3, column 1  
  - They form a rectangle → swap columns: `C F`  
- Digraph: `LX`  
  - `L` is row 4, column 1  
  - `X` is row 5, column 4 → Rectangle → `S U`  

- Ciphertext for `HELLO`: `CF SU...`  


Step 4: Decrypting

To decrypt, reverse the rules:

1. **Same row:** Use the letter immediately to the **left**.  
2. **Same column:** Use the letter immediately **above**.  
3. **Rectangle:** Swap columns back.  


## 1.3 Development in the 20th Century 


## Mechanization & Early Innovations (1900s–1940s)

### Rotor Machines and the One‑Time Pad
- **Vernam Stream Cipher & One‑Time Pad (1917)** — Gilbert S. Vernam designed an additive stream cipher combining plaintext with a key via XOR, later enhanced with truly random key tape, creating the first automatic one‑time pad system.
- **Rotor Machine Patents (1919)** — Edward Hebern developed and patented the first rotor-based cipher machine mechanism, later mirrored by inventors like Scherbius and Koch.

### Cryptanalysis and War‑Driven Breakthroughs
- **Zimmermann Telegram (1917)** — Its decryption contributed to the U.S. entering World War I.
- **Cryptanalytic Innovations** — William and Elizebeth Friedman introduced statistical methods (coincidence counting) for cryptanalysis.
- **Enigma & Lorenz Breakthroughs** — The Polish cryptanalyst Marian Rejewski first cracked Enigma ciphers (circa 1932), and Alan Turing, with team via the Bombe and later the Colossus, decrypted German Lorenz messages during WWII.


<div style="display: flex; gap: 10px; align-items: center;">
  <img src="https://upload.wikimedia.org/wikipedia/en/8/87/The_Imitation_Game_%282014%29.png" alt="Image 1" width="200" />
  <img src="https://upload.wikimedia.org/wikipedia/commons/6/66/Alan_Turing_%281951%29_%28crop%29.jpg" alt="Image 2" width="200" />
</div>

### Foundations of Modern Theory
- **Claude Shannon's Legacy** — In 1948–1949, Claude Shannon published *"Communication Theory of Secrecy Systems,"* establishing the theoretical foundations of modern cryptography—transforming it from art to science. He proved conditions for perfect secrecy and laid groundwork for symmetric systems like DES and AES.

### Rise of Modern Cryptography (1970s–1990s)

### Symmetric‑Key Evolution: DES and AES
- **DES (Data Encryption Standard)** — IBM developed DES in the mid‑1970s; it was adopted as a U.S. federal standard in 1977.
- **AES (Advanced Encryption Standard)** — In 2001, AES based on the Rijndael algorithm was selected by NIST to replace DES, addressing its vulnerability to brute force.

### Public‑Key Cryptography
- **GCHQ’s Early Work (1973–1975)** — Ellis, Cocks, and Williamson pioneered public‑key encryption in the UK, but their work remained classified until 1997.
- **Diffie–Hellman and RSA (1976–1977)** — Diffie and Hellman introduced the public-key exchange in *"New Directions in Cryptography"* (1976); RSA followed soon after in 1977, enabling digital signatures and secure key exchange.

### Civil Liberties & Crypto Activism
- **Cypherpunk Movement** — Led by figures like Phil Zimmermann (creator of PGP), the 1990s saw activists advocating for strong encryption as a basic privacy right, resisting government controls.

### New Algorithms and Techniques
- **ECC, Zero‑Knowledge, and Blind Signatures**
  - **Elliptic Curve Cryptography (1985)** — Proposed by Koblitz and Miller as a more efficient alternative to RSA/ECC.
  - **Zero‑Knowledge Proofs (1985)** — Introduced by Goldwasser, Micali, and Rackoff, allowing one party to prove knowledge without revealing the information.
  - **Blind Signatures & Digital Cash (1983)** — Developed by David Chaum to enable anonymous electronic payments.

### Cryptography’s Future

- **Quantum Key Distribution (BB84, 1984)** — Bennett and Brassard introduced protocols for quantum cryptography, laying the groundwork for theoretically secure key exchange based on physics.
- **Shor’s Algorithm (1994)** — Demonstrated quantum computers could efficiently factor integers, threatening RSA and ECC.
- **Post‑Quantum Cryptography** — In response, researchers are developing quantum-resistant algorithms to secure systems before powerful quantum machines arrive.


### Timeline 

| Period           | Key Developments |
|------------------|------------------|
| Early 1900s      | Vernam cipher, one‑time pad; rotor machines (Hebern). |
| WWI–WWII         | Zimmermann Telegram, Friedman’s cryptanalysis; Enigma and Lorenz ciphers broken; Colossus. |
| 1948–1949        | Shannon’s cryptographic theory transforms the field. |
| 1970s            | DES adopted; public-key cryptography (GCHQ, Diffie‑Hellman, RSA). |
| 1980s–1990s      | ECC, zero‑knowledge proofs, blind signatures, PGP revolutionize digital security. |
| Mid‑1990s        | Shor’s algorithm reveals quantum threats. |
| 2000s–Present    | AES replaces DES; post‑quantum cryptography and quantum key distribution emerge. |


---

# 2. Basic Concepts of Cryptography

## 2.1 Definition of Cryptography
Cryptography is the science of securing information and communications through the use of mathematical techniques. It ensures **confidentiality**, **integrity**, **authentication**, and **non-repudiation** of data. In simple terms, cryptography converts readable data (**plaintext**) into an unreadable format (**ciphertext**) to prevent unauthorized access. Only authorized parties can convert it back to the original form using a secret key.

## 2.2 Key Terms and Concepts
Understanding the following key terms is essential in cryptography:

- **Plaintext**: The original readable message or data before encryption.
- **Ciphertext**: The scrambled or unreadable form of the plaintext after encryption.
- **Encryption**: The process of converting plaintext into ciphertext using an algorithm and a key.
- **Decryption**: The reverse process of converting ciphertext back into plaintext.
- **Key**: A piece of information used by cryptographic algorithms to encrypt and decrypt data.
- **Cipher**: The algorithm used for performing encryption and decryption.
- **Symmetric Key Cryptography**: Encryption where the same key is used for both encryption and decryption.
- **Asymmetric Key Cryptography**: Encryption that uses a pair of keys — a public key for encryption and a private key for decryption.
- **Hash Function**: A function that converts data into a fixed-size hash value, often used for integrity checks.
- **Digital Signature**: A cryptographic technique to ensure the authenticity and integrity of a message.

## 2.3 Cryptography and Cryptanalysis
- **Cryptography** is the practice and study of techniques for securing information against adversaries.
- **Cryptanalysis** is the art of breaking cryptographic systems and algorithms, aiming to uncover plaintext or keys without authorization.

Both disciplines are interconnected:
- **Cryptographers** design strong algorithms and protocols.
- **Cryptanalysts** attempt to break them, which helps improve security.


---

# 3. The Role of Cryptography in Data Protection

## 3.1 Information Security and Its Components
Information security (InfoSec) refers to the practice of protecting data from unauthorized access, disclosure, alteration, and destruction. It ensures that information remains secure throughout its lifecycle. The main components of information security are:

- **Confidentiality**  
  Ensures that information is accessible only to those with proper authorization.  
  *Example:* Encryption of sensitive data to prevent unauthorized viewing.

- **Integrity**  
  Guarantees that information is accurate, consistent, and has not been altered without authorization.  
  *Example:* Hash functions and checksums to detect changes in data.

- **Availability**  
  Ensures that information and systems are available to authorized users when needed.  
  *Example:* Redundant systems and backups to prevent downtime.

- **Authentication**  
  Confirms the identity of users or systems before granting access.  
  *Example:* Use of passwords, biometrics, or multi-factor authentication.

- **Non-repudiation**  
  Prevents parties from denying their actions, ensuring accountability.  
  *Example:* Digital signatures to prove the origin of a message.

## 3.2 Objectives of Cryptography
The primary objectives of cryptography align with the principles of information security:

- **Confidentiality**  
  Protect data from unauthorized access by converting it into an unreadable format (ciphertext).  
  *Technique:* Encryption.

- **Integrity**  
  Ensure that data has not been modified during storage or transmission.  
  *Technique:* Cryptographic hash functions.

- **Authentication**  
  Verify the identity of the sender and receiver of the information.  
  *Technique:* Digital certificates, challenge-response protocols.

- **Non-repudiation**  
  Provide proof of origin and delivery, preventing denial of sending or receiving data.  
  *Technique:* Digital signatures.


# 4. Concept and Types of Ciphers

A **cipher** is an algorithm used for performing encryption and decryption — transforming plaintext into ciphertext and vice versa. Ciphers form the foundation of cryptographic systems and can be classified in several ways.

## 4.1 Symmetric and Asymmetric Ciphers

### Symmetric Ciphers
- Use the **same key** for both encryption and decryption.
- Faster and efficient for large amounts of data.
- Require secure key distribution between parties.
- **Examples:**  
  - AES (Advanced Encryption Standard)  
  - DES (Data Encryption Standard)  
  - 3DES (Triple DES)

### Asymmetric Ciphers
- Use **two different keys**: a **public key** for encryption and a **private key** for decryption.
- Simplifies key distribution but is slower than symmetric encryption.
- Commonly used for secure key exchange and digital signatures.
- **Examples:**  
  - RSA (Rivest–Shamir–Adleman)  
  - ECC (Elliptic Curve Cryptography)  
  - ElGamal

## 4.2 Structural Approaches to Classification
Ciphers can also be classified based on their internal structure and operation:

- **Stream Ciphers**  
  Encrypt data one bit or byte at a time.  
  *Example:* RC4

- **Block Ciphers**  
  Encrypt data in fixed-size blocks (e.g., 64 or 128 bits).  
  *Examples:* AES, DES

- **Substitution Ciphers**  
  Replace elements of plaintext with other elements.  
  *Example:* Caesar Cipher

- **Transposition Ciphers**  
  Rearrange the positions of characters in the plaintext.  
  *Example:* Rail Fence Cipher

- **Product Ciphers**  
  Combine substitution and transposition to increase security.  
  *Example:* Modern block ciphers like AES

## 4.3 Examples of Main Types

| Cipher Type            | Example         | Description                                  |
|------------------------|---------------|----------------------------------------------|
| **Substitution**      | Caesar Cipher | Each letter shifted by a fixed number       |
| **Transposition**     | Rail Fence    | Characters are permuted according to a key  |
| **Stream Cipher**     | RC4           | Encrypts data stream bit by bit             |
| **Block Cipher**      | AES           | Operates on fixed-size blocks of data       |
| **Asymmetric Cipher** | RSA           | Uses key pairs for encryption and decryption|



---

# 5. Cipher Requirements – Kerckhoffs's Principle

Kerckhoffs's Principle is a fundamental concept in modern cryptography that defines how a secure cryptographic system should be designed.

## 5.1 Statement of the Principle
Formulated by Auguste Kerckhoffs in 1883, the principle states:

> *A cryptographic system should remain secure even if everything about the system, except the key, is public knowledge.*

In other words:
- **Security should depend only on the secrecy of the key**, not on the secrecy of the algorithm.
- The algorithm should be publicly analyzed and tested to ensure robustness.

This principle is often summarized as:  
**“The enemy knows the system.”**

## 5.2 Importance of the Principle for Modern Systems
Kerckhoffs's Principle is crucial because:

- **Transparency Increases Security**  
  Public algorithms can be tested and improved by experts worldwide, reducing hidden flaws.
  
- **Resistance to Reverse Engineering**  
  If security depends only on the key, even if the algorithm is exposed (through leaks or reverse engineering), the system remains secure.

- **Standardization**  
  Many modern cryptographic standards (AES, RSA, SHA) are publicly known and widely analyzed.

- **Key Management Focus**  
  Efforts can be concentrated on protecting the key, which is smaller and easier to manage than an entire algorithm.

- **Defense Against “Security by Obscurity”**  
  Relying on secret algorithms is dangerous because once exposed, the system becomes completely insecure.

---

# 6. The Perfect Cipher and Classes of Cipher Strength

## 6.1 The Perfect Cipher: Concept and Examples
A **perfect cipher** is a cryptographic system in which the ciphertext provides **no information** about the original plaintext without the key. Even with unlimited computational resources, the message cannot be decrypted without the correct key.

### Characteristics of a Perfect Cipher:
- Every possible plaintext is equally likely given a ciphertext.
- Requires a key that is **truly random**, **at least as long as the message**, and **used only once**.
- Achieves **information-theoretic security**, meaning its security does not rely on computational limits.

### Example:
- **One-Time Pad (OTP)**  
  - Invented by Gilbert Vernam and Joseph Mauborgne.
  - Uses a random key that is as long as the message.
  - If used correctly (random key, no reuse), OTP is **unbreakable**.
  
**Drawback:**  
The main challenge is key distribution and secure management, which makes OTP impractical for most real-world applications.


## 6.2 Classes of Cipher Strength
Ciphers can be classified based on their level of security:

- **Unconditionally Secure (Information-Theoretic Security)**  
  Security does not depend on computational power; cannot be broken even with infinite resources.  
  *Example:* One-Time Pad.

- **Computationally Secure**  
  Cannot be broken in feasible time with current technology, but theoretically breakable with unlimited resources.  
  *Example:* AES, RSA.

- **Provably Secure**  
  Security can be formally proven based on certain mathematical assumptions (e.g., factoring large integers is hard).

- **Conditionally Secure**  
  Security depends on conditions like key secrecy and algorithm strength, but no formal proof exists.  
  *Most modern cryptosystems fall here.*

- **Broken or Weak Ciphers**  
  Vulnerable to attacks due to small key size, poor design, or discovered weaknesses.  
  *Example:* DES (due to short key length).


## 6.3. Vernam Cipher (One-time pad)

### Introduction
The **Vernam cipher** is a symmetric stream cipher invented by Gilbert Vernam in 1917. It is considered one of the earliest examples of **stream ciphers** and forms the basis for the concept of the **One-Time Pad** (OTP), which is theoretically unbreakable if implemented correctly.

### Working Principle
The Vernam cipher uses the **bitwise XOR operation** between:
- **Plaintext (P)**: The original message represented in binary.
- **Key (K)**: A sequence of random bits (key stream) of the same length as the plaintext.

**Encryption**

$$
C = P \oplus K
$$

Where:

- $C$ = Ciphertext
- $\oplus$ = XOR (exclusive OR)

**Decryption**

$$
P = C \oplus K
$$

Because:

$$
(P \oplus K) \oplus K = P
$$

**Characteristics**

- **Type**: Symmetric stream cipher.
- **Key Length**: Same length as the message.
- **Operation**: Bitwise XOR.
- **Security**: 
  - If the key is truly random, used only once, and kept secret → becomes a **One-Time Pad** → **unconditionally secure**.
  - If the key is reused or not random → cipher becomes **vulnerable**.

**Example**
```
Plaintext: `HELLO`  
Convert to binary (ASCII):  
`H = 01001000`  
`E = 01000101`  
... and so on.

Key: `10110110`  
Perform XOR:

Plaintext: 01001000
Key: 10110110
Ciphertext: 11111110
```

***Advantages and Disadvantages***

Advantages:
- Simple to implement.
- Forms the basis for the strongest cipher (One-Time Pad).

Disadvantages:
- Requires key as long as the message.
- Key must be **truly random** and **never reused**.
- Key distribution is difficult.
