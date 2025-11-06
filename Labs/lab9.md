# Assignment 9. Elliptic Curve Cryptography

## Question 1 — Elliptic Curve over $F_{43}$

Consider the elliptic curve $y^2 = x^3 + x$ over the finite field $\mathbb{F}_{43}$.

- 1.1 List all affine points (all points except the point at infinity) of the curve and plot them on a discrete grid.

- 1.2 Compute the point sum $(4, 5) + (5, 1)$ on $E(\mathbb{F}_{43})$.


## Question 2 — Elliptic Curve Digital Signature Algorithm

Consider the message: ```Write me: example@example.com``` (substitute by your email).

1. Download the private key [here](https://github.com/yefimov-d/Cryptography/blob/master/Lectures/lec9/privkey.der).
2. Compute the SHA-256 hash of the message.
3. Sign the hash using ECC with the DSS scheme in [deterministic](https://datatracker.ietf.org/doc/html/rfc6979) mode.
4. Output the resulting signature in hex.

Example: https://pycryptodome.readthedocs.io/en/latest/src/signature/dsa.html