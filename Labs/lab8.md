# Assignment 8. Digital Signatures. Message Authentication Codes.

## Question 1 — RSASSA-PKCS1-v1_5

Download the [image](https://www.osadl.org/fileadmin/dam/images/tux-72.png) and the [provided RSA key files](https://github.com/yefimov-d/Cryptography/tree/master/Lectures/lec8).


Use the [RSASSA-PKCS1-v1_5](https://pycryptodome.readthedocs.io/en/latest/src/signature/pkcs1_v1_5.html) signature scheme with SHA-256 as the hashing algorithm. Compute the SHA-256 hash of the image, sign it using the private key, and output the signature in Base64 format. Then verify the signature using the public key.


Next, make a small modification to the image file and repeat the verification step. Did the verification fail as expected?

## Question 2 — HMAC

Take the image file from the previous question. Use the [HMAC](https://pycryptodome.readthedocs.io/en/latest/src/hash/hmac.html) algorithm with SHA-256 as the underlying hash function to protect the file’s integrity. Use the following 32-byte secret key:

```5c8cd96c1749adb5f9ba379cfc546ec90d2b82dc927f6067dd444fb20e73ef32```

Compute the HMAC of the file and output its value in hexadecimal format. Next, modify the image file slightly and verify the HMAC again. Did the verification fail as expected?

> Explain the difference between digital signatures and MACs, focusing on key type, typical use cases, and non-repudiation.