# Assignment 1. Mathematical foundations of cryptography and steganography

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

## Question 1 — Factorization

**Given four integers:**

$$
72; 95; 126; 143.
$$

1) Factor each number into primes.  
2) For each number \(n_i\), find its largest prime divisor $P(n_i)$.  
3) Compute the sum

$$
S=\sum_{i=1}^{4} P(n_i).
$$

_Leave your final answer as a single integer._

---

## Question 2 — Modular calculations

Take the prime $p=13$ and

$$
\begin{aligned}
&a_1=4,\quad b_1=5,\quad c_1=6,\\
&a_2=7,\quad b_2=8,\quad c_2=9,\\
&a_3=10,\quad b_3=11,\quad c_3=12.
\end{aligned}
$$

1. Take the **first letter** of your cleaned email.  
2. Let `k` be the decimal **ASCII value** of this letter

Calculate the value of $S$:

$$ S \equiv (a_1\cdot b_1^{\,p-1}\cdot c_1) \cdot k + 
(a_2^{\,p-1}\cdot b_2\cdot c_2) \cdot k^2
      + (a_3\cdot b_3^{\,p-1}\cdot c_3) \cdot k^3 \pmod{p} $$

> Example: If the first letter is `a`, then `k = 97` (ASCII).

---

## Question 3 — Euler's Totient Function

1. Take the **last letter** of your cleaned email.  
2. Let `m` be the decimal **ASCII value** of this letter.  
3. Calculate **φ(m)**, where φ is **Euler's totient function**.

> Example: If the last letter is `e`, then `m = 101` (ASCII). Then compute φ(101).

---

## Question 4 — Extended Euclidean Algorithm

1. Take `k` (see Questions 2).  
2. Use the **Extended Euclidean Algorithm** to find:

$$
\text{GCD}(k, 257) \text{ and } k^{-1} \mod 257 \text{ (if it exists).}
$$

---

## Question 5 — Modular Exponentiation
 
1. Take `k` and `m` (see Questions 2 and 3 for these values).  
2. Compute the modular exponentiation:

$$
(k \cdot m + 1)^{\phi(m)} \mod m
$$

---

## Question 6 — Finite Field Arithmetic


In AES, bytes are interpreted as elements of the finite field 

$$
\mathbb{F}_{2^8} = \mathbb{F}_2[x] \,/\, (x^8 + x^4 + x^3 + x + 1).
$$

That is, each byte is treated as a polynomial over \(\mathbb{F}_2\), and multiplication is performed modulo the irreducible polynomial

$$
p(x) = x^8 + x^4 + x^3 + x + 1.
$$


**Task.** Take two given values `k` and `m` (as defined in Questions 2 and 3 above).  Treat them as bytes (elements of $\mathbb{F}_{2^8}$) and compute $k \cdot m$ in AES field arithmetic. Provide your answer in decimal form.