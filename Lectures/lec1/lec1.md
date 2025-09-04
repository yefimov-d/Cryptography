# Topic 1. Mathematical foundations of cryptography and steganography

## Lecture Plan
1. Prime numbers, the factorization problem, and why they matter for cryptography
2. Modular arithmetic
3. Fermat's Little Theorem, Euler Theorem, Euclidean Algorithm
4. Binary and Hexadecimal Representations of Bytes
5. Computations in Finite Fields

---


# 1. Prime numbers, the factorization problem, and why they matter for cryptography

## 1.1 What is a prime number?
A **prime number** is an integer greater than 1 whose only positive divisors are 1 and itself.  
Examples: $2, 3, 5, 7, 11, 13, \dots$

The **Fundamental Theorem of Arithmetic** states that every integer $n>1$ can be written uniquely (up to order) as a product of prime powers:

$$
n = p_1^{\alpha_1} p_2^{\alpha_2} \cdots p_k^{\alpha_k}.
$$

That factorization is unique, which makes primes the “atoms” of the integers.

**Theorem (Euclid):**  
There are infinitely many prime numbers.  In other words, the set of prime numbers is infinite.

### The integer factorization problem
Given a composite integer $N$, the **integer factorization problem** asks: find nontrivial integers $p, q > 1$ such that

$$
N = p \cdot q \cdot \dots
$$

(typically we want the prime factors). For large $N$ (hundreds or thousands of bits), this is computationally difficult in practice.

- **Easy cases:** small numbers, numbers with small prime factors, or numbers with special structure.  
- **Hard cases:** large semiprimes (product of two large primes) with no special structure are believed to be hard to factor.



### Known factorization algorithms (brief overview)
Algorithms differ greatly in applicability and complexity:

- **Trial division:** try dividing by small primes — trivial but exponentially slow for large $N$.  
- **Pollard’s rho:** a randomized algorithm good for finding small factors quickly.  
- **Elliptic Curve Method (ECM):** effective for finding medium-size prime factors.  
- **Quadratic Sieve (QS):** good for numbers up to a few hundred bits.  
- **General Number Field Sieve (GNFS):** the asymptotically fastest classical algorithm for factoring large general integers (sub-exponential time).

No deterministic polynomial-time classical algorithm for factoring is known. Factoring is **not proven** to be in P nor NP-hard — its precise complexity classification remains unresolved.



## 1.2 Why factorization matters for cryptography

### RSA and related schemes
The security of **RSA** directly relies on the assumed hardness of factoring:

- **Key generation:** choose two large random primes $p$ and $q$, compute $N = p\cdot q$. Publish $N$ (and a public exponent); keep $p,q$ secret.  
- **Security claim:** given only $N$, recovering the private key requires factoring $N$ (or otherwise deriving $\varphi(N)$ ), which is assumed infeasible for properly chosen sizes.

If an attacker factors $N$, they can compute $\varphi(N)=(p-1)(q-1)$ and derive the private exponent — thus breaking RSA.

Other schemes (e.g., the Rabin cryptosystem or constructions using Blum integers) also rely on factorization-related hardness assumptions.

### Practical consequences
- **Key sizes:** To make factoring infeasible with current classical algorithms, practice uses large RSA keys (e.g., 2048 bits or larger).  
- **Performance trade-offs:** Large keys increase computation and bandwidth costs; this is why elliptic-curve cryptography (ECC), based on different hardness assumptions, became popular — ECC gives similar security with much smaller keys.


### The quantum threat
A sufficiently large quantum computer running **Shor’s algorithm** would factor integers in polynomial time, which would **break RSA and many other schemes** based on factoring or discrete logarithms. This motivates:

- Research in **post-quantum cryptography** (algorithms believed to be secure even against quantum attackers).  
- Migration planning by industry and governments to prepare for a post-quantum world.

Why haven't quantum computers factored 21 yet: https://algassert.com/post/2500



### Why primes are chosen carefully
- **Random large primes:** Practical RSA primes are chosen randomly and tested with fast probabilistic primality tests (e.g., Miller–Rabin).  
- **Avoid weak structure:** Primes with special patterns, small factors, or shared factors across keys make factoring easier — good key generation avoids these pitfalls.  
- **Primality testing vs. factoring:** Determining whether a number is prime is relatively easy (fast tests exist); finding its factorization can remain hard.


### Intuition for hardness
Multiplying two large primes is easy; reversing the operation (factoring their product) currently has no known efficient classical shortcut for general case. Best classical factoring algorithms are sub-exponential but still infeasible for sufficiently large, well-chosen keys. Cryptography exploits this asymmetry: **easy to compute forward, hard to invert**.


---

# 2. Modular arithmetic

## 2.1 Definition

### Division with Remainder

Dividing an integer $a$ by a natural number $b > 0$ means expressing it as:

$$
a = b \cdot q + r, \quad 0 \le r < b, \quad q \in \mathbb{Z}
$$

where $q$ is the quotient and $r$ is the remainder.  

If $r = 0$, then $a$ is divisible by $b$, denoted $a \, \vdots \, b$.  
$b$ is called a divisor of $a$.

**Example:** Divide $a = 17$ by $b = 5$:

$$
17 = 5 \cdot 3 + 2
$$

Here, $q = 3$ and $r = 2$.

### Congugancy

Let $a, b$ be integers, and $m$ be a natural number. We say that $a$ and $b$ are **congruent modulo** $m$ if their difference is divisible by $m$, that is:  

$$
a \equiv b \pmod{m}
$$

means that there exists an integer $k$ such that  

$$
a - b = k \cdot m.
$$

![1](https://upload.wikimedia.org/wikipedia/commons/a/a4/Clock_group.svg)

**Example**

$$
17 \equiv 5 \pmod{6}
$$

since  

$$
17 - 5 = 12,
$$

and $12$ is divisible by $6$.




## 2.2 Properties of Congruence

The properties that hold for equality also hold for congruence modulo. Specifically, for any integers $a, b, c$ and natural number $m$, the following properties hold:

- **Reflexivity:** 

$$
a \equiv a \pmod{m}
$$

- **Symmetry:**  
If $a \equiv b \pmod{m}$, then $b \equiv a \pmod{m}$.

- **Transitivity:**  
If $a \equiv b \pmod{m}$ and $b \equiv c \pmod{m}$, then $a \equiv c \pmod{m}$.

Additionally, if $a_1 \equiv b_1 \pmod{m}$ and $a_2 \equiv b_2 \pmod{m}$, then the following properties hold:

$$
(a_1 + a_2) \equiv (b_1 + b_2) \pmod{m}, \quad
(a_1 - a_2) \equiv (b_1 - b_2) \pmod{m}, \quad
(a_1 \cdot a_2) \equiv (b_1 \cdot b_2) \pmod{m}.
$$


### Examples

1. **Example 1**  

$$
(3 + 5 \cdot 4 - 2) \mod 7 = (3 + 20 - 2) \mod 7 = 21 \mod 7 = 0
$$  

That is:  

$$
3 + 5 \cdot 4 - 2 \equiv 0 \pmod{7}
$$

2. **Example 2**  

$$
(2^3 + 4 \cdot 5) \mod 6 = (8 + 20) \mod 6 = 28 \mod 6 = 4
$$  

That is:

$$
2^3 + 4 \cdot 5 \equiv 4 \pmod{6}
$$

3. **Example 3**  

$$
(4^3 + 2 \cdot 7) \mod 5 = (64 + 14) \mod 5 = 78 \mod 5 = 3
$$  

That is:  

$$
4^3 + 2 \cdot 7 \equiv 3 \pmod{5}
$$

4. **Example 4**

$$
(5^2 + 3 \cdot 4 - 7^2) \mod 11 = (25 + 12 - 49) \mod 11 = (-12) \mod 11 = -12 + 11 = -1 \equiv 10 \pmod{11}
$$  

That is:  

$$
5^2 + 3 \cdot 4 - 7^2 \equiv 10 \pmod{11}
$$


### Ring of Residues

The ring of residues modulo $n$, denoted by $\mathbb{Z}_n$, is the set of residue classes

$$
\mathbb{Z}_n = \{ \overline{0}, \overline{1}, \dots, \overline{n-1} \}
$$

together with addition and multiplication modulo $n$:

$$
\overline{a} + \overline{b} = \overline{(a + b) \mod n}, \quad 
\overline{a} \cdot \overline{b} = \overline{(a \cdot b) \mod n}.
$$

**Example: $\mathbb{Z}/5\mathbb{Z}$**

Elements: $\{0,1,2,3,4\}$. Operations are taken modulo $5$.

Addition $(+)$

| +   | 0 | 1 | 2 | 3 | 4 |
|:---:|:-:|:-:|:-:|:-:|:-:|
| **0** | 0 | 1 | 2 | 3 | 4 |
| **1** | 1 | 2 | 3 | 4 | 0 |
| **2** | 2 | 3 | 4 | 0 | 1 |
| **3** | 3 | 4 | 0 | 1 | 2 |
| **4** | 4 | 0 | 1 | 2 | 3 |

Multiplication $(\times)$

| ×   | 0 | 1 | 2 | 3 | 4 |
|:---:|:-:|:-:|:-:|:-:|:-:|
| **0** | 0 | 0 | 0 | 0 | 0 |
| **1** | 0 | 1 | 2 | 3 | 4 |
| **2** | 0 | 2 | 4 | 1 | 3 |
| **3** | 0 | 3 | 1 | 4 | 2 |
| **4** | 0 | 4 | 3 | 2 | 1 |


## 2.3 Greatest Common Divisor (gcd)

Let $a$ and $b$ be two integers. The **greatest common divisor** (abbreviated **gcd**) of $a$ and $b$ is denoted by $\gcd(a, b)$ and is defined as the largest integer that divides both $a$ and $b$.

$$
\gcd(a, b) = \max\{d \mid d \text{ divides both } a \text{ and } b\}.
$$

### Properties of gcd

1. If $\gcd(a, b) = 1$, then $a$ and $b$ are **coprime**, i.e., they have no common divisors other than $1$.
2. For any integers $a$ and $b$:

$$
\gcd(a, b) = \gcd(b, a \mod b).
$$

This is the basis of the Euclidean algorithm for computing the gcd.

### Example

Let $a = 56$ and $b = 98$. Then:

$$
\gcd(56, 98) = 14.
$$

Since $14$ is the largest number that divides both $56$ and $98$.

## 2.4 Multiplicative Inverse Modulo

Let a modulus $m$ and an integer $a$ be given. A number $b$ is called the **multiplicative inverse** of $a$ modulo $m$ if the following holds:

$$
a \cdot b \equiv 1 \pmod{m}.
$$

That is, the product of $a$ and $b$ leaves a remainder of $1$ when divided by $m$.

### Existence of the Inverse

The inverse $b$ exists if and only if $a$ and $m$ are **coprime**, i.e.:

$$
\gcd(a, m) = 1.
$$

If $\gcd(a, m) > 1$, then $a$ does not have an inverse modulo $m$.

### Inverse for a Composite Modulus

If the modulus $m$ is composite (i.e., has nontrivial divisors), the inverse exists only for those $a$ that do not share common divisors with $m$ other than $1$. For example, if $m = 12$:

- $5$ has an inverse since $\gcd(5, 12) = 1$.
- $4$ does not have an inverse since $\gcd(4, 12) = 4 \neq 1$.

If the modulus $m$ is a **prime number**, the inverse exists for all nonzero elements $a$ (i.e., for all $1 \leq a < m$).

### Inverse for a Prime Modulus

If $m$ is a **prime number**, any nonzero element $a$ has an inverse. In this case, the inverse can be found using the extended Euclidean algorithm or Fermat's little theorem:

$$
a^{m-2} \equiv a^{-1} \pmod{m}.
$$

### Example

Find the inverse of $3$ modulo $7$. We need $b$ such that:

$$
3 \cdot b \equiv 1 \pmod{7}.
$$

By checking possible values, we find:

$$
3 \cdot 5 = 15 \equiv 1 \pmod{7}.
$$

Thus:

$$
3^{-1} \equiv 5 \pmod{7}.
$$


---

# 3. Fermat's Little Theorem, Euler Theorem, Euclidean Algorithm

## 3.1 Fermat's Little Theorem

Let $p$ be a prime number and $a$ an integer such that $a$ is not divisible by $p$ (i.e., $\gcd(a, p) = 1$). Then, by Fermat's little theorem:

$$
a^{p-1} \equiv 1 \pmod{p}.
$$

This means that if $p$ is prime and $a$ is not divisible by $p$, then $a$ raised to the power of $p-1$ leaves a remainder of $1$ when divided by $p$.

### Example

Let $p = 7$ and $a = 2$. Then, by Fermat's little theorem:

$$
2^{7-1} = 2^6 = 64 \equiv 1 \pmod{7}.
$$

Thus, $2^6 \equiv 1 \pmod{7}$, confirming Fermat's theorem in this case.


## 3.2 Euler's Theorem

Let $a$ and $n$ be integers, where $n$ is a natural number and $\gcd(a, n) = 1$. Then, by Euler's theorem:

$$
a^{\varphi(n)} \equiv 1 \pmod{n},
$$

where $\varphi(n)$ is Euler's totient function, which counts the numbers less than $n$ that are coprime to $n$.

Euler's function $\varphi(n)$ for any natural number $n$ is calculated as follows:

If 

$$
n = p_1^{k_1} \cdot p_2^{k_2} \cdot \dots \cdot p_m^{k_m}
$$ 

is the prime factorization of $n$, then

$$
\varphi(n) = n \cdot \left( 1 - \frac{1}{p_1} \right) \cdot \left( 1 - \frac{1}{p_2} \right) \cdot \dots \cdot \left( 1 - \frac{1}{p_m} \right) = \prod_{i=1}^{m} (p_i^{k_i} - p_i^{k_i - 1}).
$$



### Example

Let $a = 3$ and $n = 10$. Then $\gcd(3, 10) = 1$, and Euler's totient function for $n = 10$ is:

$$
\varphi(10) = 10 \cdot \left( 1 - \frac{1}{2} \right) \cdot \left( 1 - \frac{1}{5} \right) = 10 \cdot \frac{1}{2} \cdot \frac{4}{5} = 4.
$$

Thus, by Euler's theorem:

$$
3^4 \equiv 1 \pmod{10}.
$$

Check:

$$
3^4 = 81 \equiv 1 \pmod{10}.
$$

This confirms Euler's theorem for this example.


## 3.3 Euclidean Algorithm

The Euclidean algorithm is an efficient method for finding the greatest common divisor (GCD) of two natural numbers.  
It is based on the following recursive relation:

$$
\gcd(a, b) =
\begin{cases} 
b, & \text{if } a \mod b = 0, \\
\gcd(b, a \mod b), & \text{otherwise}.
\end{cases}
$$

where $a \mod b$ is the remainder of $a$ divided by $b$.  
The process repeats until one of the numbers becomes zero, at which point the other number is the GCD.


### Euclidean Algorithm: GCD and Modular Inverse

The Euclidean algorithm allows finding the greatest common divisor (GCD) of two numbers $a$ and $b$ by repeatedly computing remainders:

$$
\gcd(a, b) = \gcd(b, a \mod b).
$$

If $\gcd(a, n) = 1$, then a modular inverse $a^{-1}$ exists, which can be found using the extended Euclidean algorithm by solving:

$$
ax + ny = 1.
$$

Then $x \pmod{n}$ is the modular inverse $a^{-1}$.





### Example: Find $\gcd(26, 11)$ and $11^{-1} \mod 26$

Apply the Euclidean algorithm:

$$
\begin{aligned}
26 &= 2 \cdot 11 + 4 \\
11 &= 2 \cdot 4 + 3 \\
4  &= 1 \cdot 3 + 1 \\
3  &= 3 \cdot 1 + 0
\end{aligned}
$$

So:

$$
\gcd(26, 11) = 1
$$


Since $\gcd(26, 11) = 1$, the inverse exists.

The extended Euclidean algorithm gives the equations:

$$
\begin{aligned}
1 &= 4 - 1\cdot 3 \\
  &= 4 - 1\cdot(11 - 2\cdot 4) = 3\cdot 4 - 11 \\
  &= 3\cdot(26 - 2\cdot 11) - 11 = 3\cdot 26 -7\cdot 11.
\end{aligned}
$$

So $1 = 3\cdot 26 -7\cdot 11$. Hence the coefficient of $11$ is $-7$, and

$$
11^{-1} \bmod 26 \equiv -7 \bmod 26 \equiv 19.
$$

**Check:**

$$
11\cdot 19 = 209 \equiv 1 \pmod{26}.
$$

---
# 4. Binary and Hexadecimal Representations of Bytes

A **byte** is a unit of digital information that typically consists of **8 bits**.  
Each bit can be either `0` or `1`, so a byte can represent values from `0` to `255` in decimal.


## 4.1 Binary Representation
- In **binary**, each bit of a byte is written explicitly as a sequence of 8 digits (`0` or `1`).  
- Example:  
  - Decimal `13` → Binary `00001101`  
  - Decimal `255` → Binary `11111111`  

Binary notation is very close to how computers actually store information, but it can be **long and hard to read** for humans.


## 4.2 Hexadecimal Representation
- In **hexadecimal (base 16)**, each digit represents **4 bits (a nibble)**.  
- A single byte (8 bits) can be represented by **two hexadecimal digits**.  
- Hexadecimal digits go from `0–9` and then `A–F` (where `A = 10`, ..., `F = 15`).  

### Examples:
- Decimal `13` → Binary `00001101` → Hex `0x0D`  
- Decimal `255` → Binary `11111111` → Hex `0xFF`  
- Decimal `64` → Binary `01000000` → Hex `0x40`  



### Why Use Hexadecimal?
- More **compact**: `0xFF` is shorter than `11111111`.  
- Easy **conversion**: every 4 bits map exactly to one hex digit.  
- Widely used in programming, memory dumps, cryptography, and networking (e.g., MAC addresses, IPv6).  



## 4.3  Conversion Table (Binary ↔ Hex)
| Binary | Hex | Decimal |
|--------|-----|---------|
| 0000   | 0   | 0       |
| 0001   | 1   | 1       |
| 0010   | 2   | 2       |
| 0011   | 3   | 3       |
| 0100   | 4   | 4       |
| 0101   | 5   | 5       |
| 0110   | 6   | 6       |
| 0111   | 7   | 7       |
| 1000   | 8   | 8       |
| 1001   | 9   | 9       |
| 1010   | A   | 10      |
| 1011   | B   | 11      |
| 1100   | C   | 12      |
| 1101   | D   | 13      |
| 1110   | E   | 14      |
| 1111   | F   | 15      |


### Example: RGB

- **RGB** stands for **Red, Green, Blue**.  
- It is a color model used in digital systems (monitors, images, web design).  
- Each color channel (**R, G, B**) is represented by **1 byte (8 bits)**.  

**Examples of Colors**
| Color | Decimal (R,G,B)       | Binary (R,G,B)                     | Hexadecimal |
|-------|-----------------------|-------------------------------------|-------------|
| Black | (0, 0, 0)             | `00000000 00000000 00000000`        | `#000000`   |
| White | (255, 255, 255)       | `11111111 11111111 11111111`        | `#FFFFFF`   |
| Red   | (255, 0, 0)           | `11111111 00000000 00000000`        | `#FF0000`   |
| Green | (0, 255, 0)           | `00000000 11111111 00000000`        | `#00FF00`   |
| Blue  | (0, 0, 255)           | `00000000 00000000 11111111`        | `#0000FF`   |
| Cyan  | (0, 255, 255)         | `00000000 11111111 11111111`        | `#00FFFF`   |
| Magenta | (255, 0, 255)       | `11111111 00000000 11111111`        | `#FF00FF`   |
| Yellow | (255, 255, 0)        | `11111111 11111111 00000000`        | `#FFFF00`   |

![1](https://upload.wikimedia.org/wikipedia/commons/1/11/RGBCube_b.svg)

## 4.4 ASCII

[Unicode, in friendly terms: ASCII, UTF-8, code points, character encodings, and more](https://youtu.be/ut74oHojxqo?si=e_b_IMAUdLQ5fZbI)

### What is ASCII?
- **ASCII** stands for **American Standard Code for Information Interchange**.  
- It is a character encoding standard that assigns a **number (byte)** to each character (letters, digits, symbols).  
- Standard ASCII uses **7 bits**, but it is usually stored in **1 byte (8 bits)**.  

### Binary Representation
- Each ASCII character can be represented as an **8-bit binary number**:  
- Example:  
  - Character `'A'` → Decimal `65` → Binary `01000001`  
  - Character `'a'` → Decimal `97` → Binary `01100001`  
  - Character `'0'` → Decimal `48` → Binary `00110000`  

Binary shows exactly how the computer stores the character in memory.


### Hexadecimal Representation
- Each byte can also be written as **two hexadecimal digits**.  
- Example:  
  - `'A'` → Binary `01000001` → Hex `0x41`  
  - `'a'` → Binary `01100001` → Hex `0x61`  
  - `'0'` → Binary `00110000` → Hex `0x30`  

Hexadecimal is shorter and more readable than binary for humans.


## ASCII Table Examples
| Character | Decimal | Binary     | Hex  |
|-----------|---------|-----------|------|
| NUL       | 0       | 00000000  | 0x00 |
| Space     | 32      | 00100000  | 0x20 |
| `0`       | 48      | 00110000  | 0x30 |
| `A`       | 65      | 01000001  | 0x41 |
| `a`       | 97      | 01100001  | 0x61 |
| `z`       | 122     | 01111010  | 0x7A |
| DEL       | 127     | 01111111  | 0x7F |

Full table: https://www.ascii-code.com/

### Why Hex is Useful for ASCII
- Compact representation of characters in **memory dumps, programming, and cryptography**.  
- Easy to convert back to binary or decimal.  
- Many file formats, network protocols, and cryptographic algorithms display characters in hex.


---

## 5. Computations in Finite Fields

## 5.1 Description

For a more rigorous definition, see: https://en.wikipedia.org/wiki/Field_(mathematics)

A finite field is a set of elements with two operations—addition and multiplication—that satisfy the field axioms and contain only a finite number of elements.  

The number of elements in a finite field is called its **order**. Any finite field has order $p^n$, where $p$ is a prime number and $n$ is a natural number. Such fields are denoted by $\mathbb{F}_{p^n}$.


### Arithmetic in Finite Fields (Galois Fields)

See also: https://en.wikipedia.org/wiki/Finite_field

If $n = 1$, the field $\mathbb{F}_p$ is the field of residues modulo a prime $p$, where addition and multiplication are performed modulo $p$.  

If $n > 1$, the field $\mathbb{F}_{p^n}$ is constructed using an irreducible polynomial $f(x)$ of degree $n$ over $\mathbb{F}_p$. The elements of this field can be represented as polynomials of degree less than $n$ over $\mathbb{F}_p$:

$$
a(x) = a_0 + a_1 x + a_2 x^2 + \dots + a_{n-1} x^{n-1}, \quad a_i \in \mathbb{F}_p.
$$

### Addition
Addition in $\mathbb{F}_{p^n}$ is performed component-wise modulo $p$:

$$
(a(x) + b(x)) \mod p.
$$

### Multiplication
Multiplication is performed as usual polynomial multiplication, followed by reduction modulo the irreducible polynomial $f(x)$:

$$
(a(x) \cdot b(x)) \mod f(x).
$$


### Properties
- In any finite field, every nonzero element has a multiplicative inverse.  
- If $n > 1$, multiplication is not performed simply modulo $p$ as in prime fields; it requires reduction modulo the irreducible polynomial.  
- Elements can be represented as powers of a primitive element $g$, i.e., any nonzero element can be written as $g^k$.  See also: https://en.wikipedia.org/wiki/Primitive_element_(finite_field)



## 5.2 AES Finite Field — short intro and byte representations

AES works over the finite field $\mathbb{F}_{2^8}$. This field has $256$ elements and is built from polynomials with coefficients in $\mathbb{F}_2=\{0,1\}$ of degree \< 8, reduced modulo the irreducible polynomial used by AES:

$$
f(x) = x^8 + x^4 + x^3 + x + 1.
$$

Each element of the field is represented by one **byte** (8 bits). Arithmetic in this field is:

- **Addition:** bitwise XOR (no carries) — corresponds to adding polynomials coefficient-wise mod 2.  
- **Multiplication:** carry-less polynomial multiplication followed by reduction modulo $f(x)$.


### Bytes as Elements of the AES Field

A byte can be represented in several equivalent ways:

- **Hex / Integer:** e.g. `0x57` (decimal 87).  
- **Binary (bits):** `0x57 = 0b01010111`. We usually label bits $b_7 b_6 \dots b_0$ with $b_7$ the most-significant bit (MSB).  
- **Polynomial:** interpret bits as coefficients of a polynomial of degree ≤ 7:

$$
b_7 x^7 + b_6 x^6 + \dots + b_1 x + b_0.
$$

**Convention:** bit $b_7$ corresponds to coefficient of $x^7$, $b_0$ to $x^0$.

### Examples

- `0x01 = 0b00000001` → polynomial $1$.  
- `0x02 = 0b00000010` → polynomial $x$.  
- `0x03 = 0b00000011` → polynomial $x + 1$.  
- `0x57 = 0b01010111` → polynomial $x^6 + x^4 + x^2 + x + 1$.  
- `0x83 = 0b10000011` → polynomial $x^7 + x + 1$.

So the mapping is direct: the byte `b7 b6 ... b0` ↔ polynomial $\sum_{i=0}^{7} b_i x^i$.



### Addition 

Addition in $\mathbb{F}_{2^8}$ is XOR.

Example:

$$
0x57 \oplus 0x83 = 0b01010111 \oplus 0b10000011 = 0b11010100 = 0xD4.
$$

Polynomial view: $(x^6 + x^4 + x^2 + x + 1) + (x^7 + x + 1) = x^7 + x^6 + x^4 + x^2.$



### Multiplication (idea + useful `xtime` trick)

Multiply two bytes by:

1. Interpreting them as polynomials,
2. Multiply polynomials in $\mathbb{F}_2$ (carry-less),
3. Reduce the result modulo $f(x)$.

A practical AES shortcut is the **xtime** operation — multiply by $x$ (i.e. by 0x02):

- `xtime(a)` = (left shift `a` by 1 bit) XOR `0x1B` **iff** the MSB (Most Significant Bit) of `a` was 1; otherwise just left shift.
- `0x1B` is the byte representation of the polynomial $x^4 + x^3 + x + 1$ and corresponds to reduction with $f(x)$.

Using xtime, multiplication by small constants is easy:
- multiply by `0x02` → `xtime(a)`.
- multiply by `0x03` → `xtime(a) XOR a` (because $3 = 2 + 1$ in the field).


### Why all this matters in AES

- The **S-box** in AES uses multiplicative inverses in $\mathbb{F}_{2^8}$ (plus an affine transform).  
- **MixColumns** multiplies 4-byte vectors by a fixed $4\times4$ matrix over $\mathbb{F}_{2^8}$. Implementations rely heavily on xtime and small-constant multiplies (×2, ×3, ×9, ×11, ×13, ×14) optimized with shifts and XORs or lookup tables.


### Compact reference table (few bytes)

| Hex | Binary       | Polynomial                      |
|-----|--------------|---------------------------------|
| 0x01| 00000001     | $1$                           |
| 0x02| 00000010     | $x$                           |
| 0x03| 00000011     | $x+1$                         |
| 0x57| 01010111     | $x^6 + x^4 + x^2 + x + 1$     |
| 0x83| 10000011     | $x^7 + x + 1$                 |
| 0xC1| 11000001     | $x^7 + x^6 + 1$   |




### Numerical example

- `xtime(0x57)`:
  - `0x57 = 0b01010111`, MSB = 0 → left shift: `0b10101110 = 0xAE`. So `xtime(0x57) = 0xAE`.
- `0x57 * 0x03`:
  - `xtime(0x57) XOR 0x57 = 0xAE XOR 0x57 = 0xF9`.

A classic full multiplication example (carry-less then reduce) often shown in AES references:

$$
0x57 \times 0x83 \equiv 0xC1 \quad \text{in } \mathbb{F}_{2^8}.
$$

(You can compute this either via polynomial multiplication + reduction or step-by-step using xtime sequences and XORs.)


### Step-by-step: $0x57 \times 0x83$ in $\mathbb{F}_{2^8}$

**1) Byte → polynomial**

- $0x57 = \text{binary }01010111$  

  $$
  a(x)=x^6 + x^4 + x^2 + x + 1
  $$

- $0x83 = \text{binary }10000011$  

  $$
  b(x)=x^7 + x + 1
  $$

(Convention: bit $b_i$ ↔ coefficient of $x^i$.)


**2) Multiply as polynomials (carry-less, coefficients mod 2)**

Compute partial products (shifted copies of $a(x)$) corresponding to the 1-bits of $b(x)$:

- $a(x)\cdot 1 = x^6 + x^4 + x^2 + x + 1$

- $a(x)\cdot x = x^7 + x^5 + x^3 + x^2 + x$  (shift every exponent by 1)

- $a(x)\cdot x^7 = x^{13} + x^{11} + x^9 + x^8 + x^7$ (shift by 7)

Now add (XOR) these three partial products (since arithmetic is mod 2, terms occurring twice cancel):

List exponents from each:
- from $a(x)$: 6,4,2,1,0  
- from $a(x)\cdot x$: 7,5,3,2,1  
- from $a(x)\cdot x^7$: 13,11,9,8,7

Count occurrences (mod 2):
- 13: 1  
- 11: 1  
- 9: 1  
- 8: 1  
- 7: 2 → cancels (0)  
- 6: 1  
- 5: 1  
- 4: 1  
- 3: 1  
- 2: 2 → cancels (0)  
- 1: 2 → cancels (0)  
- 0: 1

So the raw (unreduced) product polynomial is:

$$
a(x)b(x) = x^{13} + x^{11} + x^9 + x^8 + x^6 + x^5 + x^4 + x^3 + 1.
$$

(Equivalent set of exponents: $\{13,11,9,8,6,5,4,3,0\}$.)


**3) Reduce modulo the AES irreducible polynomial**
We reduce all terms with degree $\ge 8$ using

$$
f(x) = x^8 + x^4 + x^3 + x + 1 \quad\Rightarrow\quad x^8 \equiv x^4 + x^3 + x + 1 \pmod{f(x)}.
$$

A convenient systematic way: eliminate the highest-degree terms one at a time by XORing $f(x)$ shifted to match that degree.

**Step A — eliminate $x^{13}$**  
Multiply $f(x)$ by $x^{5}$ (because $13-8=5$):

$$
f(x)\cdot x^5 = x^{13} + x^9 + x^8 + x^6 + x^5.
$$

XOR this with the current polynomial to remove $x^{13}$ and toggle the listed lower terms.

Current exponents before XOR: $\{13,11,9,8,6,5,4,3,0\}$.  
XOR with $\{13,9,8,6,5\}$ → remove 13, and toggle 9,8,6,5:

New exponent set: $\{11,4,3,0\}$.

**Step B — eliminate $x^{11}$**  
Multiply $f(x)$ by $x^{3}$ (because $11-8=3$):

$$
f(x)\cdot x^3 = x^{11} + x^7 + x^6 + x^4 + x^3.
$$

XOR this with the current polynomial:

Current before XOR: $\{ 11,4,3,0 \}$  
XOR with $\{ 11,7,6,4,3 \}$ → remove 11, remove 4 and 3 (they cancel), and add 7 and 6:

New exponent set: $\{ 7,6,0 \}$.

Now all exponents are \< 8, so reduction is finished.


**4) Final polynomial → byte**

Remaining polynomial:

$$
c(x) = x^7 + x^6 + 1.
$$

Convert to bits: $b_7=1,\; b_6=1,\; b_0=1$ → binary `11000001` → hex:

$$
c = 0xC1.
$$


### Final result

$$
0x57 \times 0x83 \equiv 0xC1 \quad\text{in }\mathbb{F}_{2^8} \text{ (AES field).}
$$



### Multiplication in AES Field $F_{2^8}$ using xtime

In AES, multiplication in the finite field $\mathbb{F}_{2^8}$ can be efficiently implemented using the `xtime` method, which performs carry-less multiplication with reduction by the irreducible polynomial $x^8 + x^4 + x^3 + x + 1$ (0x11B).

https://wiki.python.org/moin/BitwiseOperators

```python
def xtime(a):
    """Multiply by x (0x02) in GF(2^8) with reduction."""
    a <<= 1
    if a & 0x100:       # if overflow past 8 bits
        a ^= 0x11B      # reduce modulo AES irreducible polynomial
    return a & 0xFF      # keep only 8 bits

def gf_mult(a, b):
    """Multiply two bytes a and b in GF(2^8)."""
    result = 0
    for i in range(8):
        if b & 1:         # if LSB of b is 1
            result ^= a   # add a to result (XOR)
        b >>= 1            # shift b to process next bit
        a = xtime(a)       # multiply a by x
    return result

# Example: multiply 0x57 by 0x83
a = 0x57
b = 0x83
c = gf_mult(a, b)
print(f"0x{a:02X} * 0x{b:02X} = 0x{c:02X}")  # Output: 0xC1
```