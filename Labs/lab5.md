# Assignment 5. Hash Functions

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

## Question 1 — Birthday paradox 

Calculate probability of at least one collision and expected (mean) number of colliding pairs:

- exactly (using the product formula) and using the approximation for 1.1 
- using only the approximation for 1.2:

 **(1.1)**  $m = 365$ (simulation of "days in a year") and $N = 23,\,50$.
 **(1.2)**  $m = 2^{160}$ (160-bit hash, ```SHA-1``` like) and $N = 10^{24}$.



---

## Question 2 — Monte Carlo experiments


Write a program that performs a Monte Carlo experiment:

- Repeat the experiment $T = 5000$ times.  
- In each run, generate $N$ random numbers from the set $\{0, \dots, m-1\}$.  
- Compute the number of colliding pairs, and  whether at least one collision occurred.  

After $T$ runs:  
- Calculate the **empirical probability of at least one collision** across experiments.  
- Calculate the **average number of colliding pairs** across experiments.  

Compare these values with the theoretical expectations: 

- $\mathbb{E}[\#\text{pairs}] = \dfrac{N(N-1)}{2m}$  
- $P_\text{coll} = 1 - \prod_{i=0}^{N-1} \left(1 - \tfrac{i}{m}\right)$ (or its approximation $1 - e^{-\mathbb{E}[\#\text{pairs}]}$).  

Use the parameters:

| Task | $m$      | $N$   |
| ---- | -------- | ----- |
| 2.1  | 365      | 23    |
| 2.2  | 365      | 50    |
| 2.3  | $2^{32}$ | 1000  |
| 2.4  | $2^{32}$ | 10000 |


---

## Question 3 — Hash Digests

**(3.1)** Take your transformed email and compute its **MD5** hash. Then replace the first letter of your email with the letter x (if x was the first letter, replace it with a) and compute the **MD5** hash again. Compare the two digests. How many bits out of 128 have changed? Read about the [Avalanche Effect](https://en.wikipedia.org/wiki/Avalanche_effect).



**(3.2)** Take the PDF file [here](https://github.com/yefimov-d/Cryptography/blob/master/Lectures/lec3/AES.pdf). Compute its hash value using **SHA-256**. Output the hash values in hexadecimal format.

Use any available tool or library, such as ```PyCryptodome```, ```OpenSSL``` or ```hashlib``` (which may use ```OpenSSL``` internally). 

---
## Optional Task: Implement Kupyna

Implement the **Kupyna cryptographic hash function**. Your implementation should:
- Support hash lengths from 8 bits up to 512 bits in 8-bit increments.
- Accept an input string or file and return the hash in **hexadecimal format**.
- Be tested on several sample inputs to verify correctness.

Test vectors can be found in the paper.

Paper: https://eprint.iacr.org/2015/885.pdf

Author's reference C implementation: https://github.com/Roman-Oliynykov/Kupyna-reference



---

## Appendix

_Assumptions._ Let a hash function map inputs uniformly at random into a set of $m$ possible values (buckets). Take $N$ independent inputs (e.g. names, messages, keys). We treat each input’s hash as an independent uniformly distributed integer in $\{0,\dots,m-1\}$.

**Probability of no collision:**

The probability that all $N$ hashes are pairwise distinct (no collision) is
$$
P_{\text{no\_coll}}(N,m)=\prod_{i=0}^{N-1}\left(1-\frac{i}{m}\right).
$$

**(Theoretical) probability of at least one collision:**

$$
P_{\text{coll}}(N,m)=1-P_{\text{no\_coll}}(N,m).
$$

**(Theoretical) expected number of colliding pairs:**

$$
\mathbb{E}[X]=\frac{N(N-1)}{2m}.
$$
This gives the average number of colliding pairs (not the probability of at least one collision).

**Exponential approximation (when $N\ll m$)**:


$$
P_{\text{coll}}(N,m)\approx 1-\exp\!\left(-\frac{N(N-1)}{2m}\right).
$$
This approximation is accurate when $\frac{N^2}{m}$ is small-to-moderate (so higher-order terms in the log expansion are negligible).

**Birthday bound (50% collision threshold)**

Set $P_{\text{coll}}\approx 0.5$ in the exponential approximation:
$$
1-\exp\!\left(-\frac{N(N-1)}{2m}\right)=\tfrac{1}{2}
\quad\Rightarrow\quad
\exp\!\left(-\frac{N(N-1)}{2m}\right)=\tfrac{1}{2}.
$$
Taking logs and approximating $N(N-1)\approx N^2$ when $N$ is moderately large,
$$
-\frac{N^2}{2m}=\ln\frac{1}{2} = -\ln 2
\quad\Rightarrow\quad
N\approx\sqrt{2m\ln 2}\approx 1.17741\sqrt{m}.
$$
Thus collisions become likely when $N$ is on the order of $\sqrt{m}$ — this is the birthday bound.

**Poisson approximation for the number of colliding pairs**

Define $\lambda=\mathbb{E}[X]=\frac{N(N-1)}{2m}$. When $\lambda$ is small (rare collisions) and pairwise collision events are approximately independent, the distribution of $X$ is well-approximated by $\operatorname{Pois}(\lambda)$. In that regime,
$\Pr(X=0)\approx e^{-\lambda}$, so $\Pr(\text{at least one collision}) \approx 1-e^{-\lambda}$ (consistent with the exponential approximation).

Even with very large $m$, collisions can appear much earlier than intuition that expects $N\sim m$. The relevant scale is $N\sim\sqrt{m}$.

For a **cryptographic hash** of $b$ bits, $m=2^b$, so the birthday bound gives collision work around $2^{b/2}$. For example, a 128-bit hash resists brute-force collisions up to about $2^{64}$ operations.

The expected number of colliding pairs $\mathbb{E}[X]$ gives a simple way to estimate how many collisions (on average) will be present in a dataset of size $N$, while $P_{\text{coll}}$ quantifies the chance of at least one collision.












