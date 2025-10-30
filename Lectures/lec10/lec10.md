# Cryptography in applied systems

## Applied Cryptography

In this lecture, we will consider key cryptographic protocols — TLS/SSL, HTTPS, SSH, and IPSec — focusing on how they use cryptographic primitives. We will also discuss key management, secure authentication, and other applications of cryptography.

Applied cryptography is a branch of cryptography focused on the implementation and use of cryptographic techniques in real-world systems. While theoretical cryptography develops the mathematical foundations and proofs of security, applied cryptography translates these ideas into practical mechanisms that secure communication, data storage, authentication, and digital interactions.

Modern information systems rely on cryptography as an essential part of their infrastructure. It is not an optional enhancement but a foundational element of secure digital ecosystems.

Cryptography underlies:

- Secure communication protocols (e.g., TLS/SSL, SSH, IPsec);
- Data protection mechanisms (e.g., disk encryption, secure cloud storage);
- Identity and authentication systems (e.g., digital signatures, certificates);
- Blockchain and distributed ledgers (e.g., consensus mechanisms, cryptographic proofs).

Applied cryptography aims to achieve four fundamental security properties that form the basis of secure systems:


| Goal | Description |
|------|--------------|
| **Confidentiality** | Ensures that information is accessible only to authorized entities. Achieved through encryption algorithms such as AES or ChaCha20. |
| **Integrity** | Protects data from unauthorized modification or corruption. Ensured via hash functions (e.g., SHA-256) and message authentication codes (MACs). |
| **Authenticity** | Verifies the identity of participants in communication or data exchange. Implemented using authentication protocols and digital signatures. |
| **Non-repudiation** | Prevents parties from denying their actions, such as sending a message or signing a document. Enabled through asymmetric cryptography and public key infrastructures (PKIs). |


## SSL/TLS

See also: https://dev.to/techschoolguru/a-complete-overview-of-ssl-tls-and-its-cryptographic-system-36pd

SSL (Secure Sockets Layer) is the **deprecated**, pre-TLS name for the family of Internet session-security protocols originally developed by Netscape; its last published version is SSL 3.0, which is now a historic document and explicitly must not be used. Industry and standards advise to disable SSL.

TLS is the successor to SSL and offers improved security and efficiency, although the term “SSL” is still often used informally to refer to both.

The primary purpose of SSL/TLS is to ensure confidentiality, integrity, and authenticity of data transmitted between a client (for example, a web browser) and a server. These goals are achieved through a combination of asymmetric and symmetric cryptography, along with hashing algorithms.

During the TLS handshake, the client and server agree on protocol versions, exchange certificates, and establish session keys. Typically, the server sends its X.509 certificate, allowing the client to verify its identity using a trusted certificate authority (CA). Once authentication is completed, both parties derive a shared session key using asymmetric encryption (such as RSA or ECDHE). This key is then used for fast symmetric encryption of data throughout the session.

SSL/TLS protects against common network threats such as eavesdropping, message tampering, and impersonation. Modern web browsers and servers primarily use TLS 1.2 or TLS 1.3, with earlier SSL versions considered insecure and deprecated.

In practice, SSL/TLS is most commonly seen in HTTPS, but it is also used in many other protocols, such as SMTPS, FTPS, and IMAPS, ensuring secure communication across a wide range of network services.

| Aspect                            |                                  TLS 1.2 | TLS 1.3                                                  |
| --------------------------------- | ---------------------------------------: | -------------------------------------------------------- |
| Handshake RTTs for new connection |                  2 round trips (typical) | 1 round trip (reduced latency)                           |
| Key-exchange modes                |    RSA, DHE, ECDHE (static RSA possible) | Ephemeral key exchange only (e.g. ECDHE)                 |
| Cipher structure                  | Many cipher/MAC combos; AEAD recommended | Only AEAD ciphersuites (AES-GCM, ChaCha20-Poly1305, ...) |
| Forward secrecy                   |       Optional (depends on key exchange) | Default (ephemeral keys)                                 |
| Complexity / attack surface       |                    Larger (more options) | Smaller (simpler, safer defaults)                        |


It is important to understand three practical points. 

**First**, TLS provides a protocol framework (handshake + record layer + PKI-based authentication) rather than a single algorithm; correct security depends on both protocol version and parameter choices. 

**Second**, forward secrecy (ephemeral key exchange) and AEAD ciphers are central to modern best practice because they limit damage when long-term keys are compromised and provide authenticated encryption respectively. 

**Third**, secure deployment requires operational hygiene: disable old protocol versions and weak ciphers, manage certificate chains and renewal, and follow current guidance from standards bodies and security communities.

## HTTP/HTTPS

**HTTP (HyperText Transfer Protocol)** is an application-layer protocol for transmitting hypertext documents and other resources over a network. It operates on top of the transport layer (usually TCP) and provides a structured mechanism for exchanging requests and responses between a client and a server.  

However, standard HTTP **does not provide confidentiality or integrity of data**: all messages are transmitted in plaintext, making them vulnerable to interception, modification, or man-in-the-middle (MITM) attacks.  

**HTTPS (HTTP Secure)** is an extension of HTTP that uses **TLS (Transport Layer Security)** to protect the communication channel. In cryptographic terms, HTTPS provides:  

The operation of HTTPS can be described as follows:  

1. The client initiates a connection to the server and receives its certificate.  
2. The certificate is verified, and encryption parameters are negotiated through the TLS handshake.  
3. After a successful handshake, the client and server use the agreed symmetric key to exchange HTTP messages securely.  

HTTPS is a **practical application of cryptographic infrastructure**, ensuring secure interactions over the global network.


## SSH

SSH (Secure Shell) is a cryptographic network protocol that provides secure remote access, command execution, and data transfer over an unsecured network. It ensures confidentiality, integrity, and authentication through encryption and key-based authentication mechanisms. SSH uses a three-layer architecture.

- Transport — establishes a secure channel (encryption, integrity, key exchange).
- Authentication — verifies the client.
- Connection — handles command execution and data transfer.

Transport layer ([RFC 4253](https://www.rfc-editor.org/rfc/rfc4253)). The main goal is to establish a shared session key for symmetric encryption and integrity verification. Key exchange is typically based on Diffie-Hellman: both sides compute a shared secret from which encryption and MAC keys are derived. To authenticate the server, a server key pair (host keys) is used. The server signs a message with its private key, and the client verifies it with the public key.
Clients store known server public keys in ```~/.ssh/known_hosts```.
If a key changes, SSH warns about a possible man-in-the-middle attack.

User-authentication layer ([RFC 4252](https://www.rfc-editor.org/rfc/rfc4252)). After server identification, the client proves its identity.
Main methods:
Password — simple but less secure.
Key-based — more secure (RSA/DSA/ECDSA, certificates). The client generates a key pair (```id_rsa, id_rsa.pub```), and the administrator adds the public key to the user’s ```~/.ssh/authorized_keys```.
The client signs a message with its private key, and the server verifies the signature.
The private key remains only with the client and is never transmitted.
The client initiates auth requests; the server answers.

Connection layer ([RFC 4254](https://www.rfc-editor.org/rfc/rfc4254)).  This layer manages actions like running commands, transferring files, and updating terminal settings. It uses keys established in the previous step.


Algorithms: 

```EdDSA```, ```ECDSA```, ```RSA```, ```DSA``` (public keys); 

```ECDH/Diffie–Hellman``` (key exchange); 

```HMAC/UMAC``` and ```AEAD``` (MACs); 

```AES```(and older ```RC4/3DES/DES```) for symmetric encryption; 

```AES-GCM``` and ```ChaCha20-Poly1305``` for AEAD; 

```SHA``` (```MD5``` deprecated) for fingerprints.


## IPSec/VPN


IPSec is a set of standards created by the IETF’s IPSec Working Group that lets two IP devices protect the traffic between them. It gives three main security services: authentication (prove who’s speaking), integrity (make sure data isn’t changed in transit), and confidentiality (keep the data private). Keys and security associations that IPSec needs can be entered by hand or negotiated automatically using the Internet 

There are two IKE versions you should know:

IKEv1 — specified in RFC 2409. It’s the older version and has been supported on z/OS® Communications Server for many years.

IKEv2 — specified in RFC 5996. Support for IKEv2 was added in z/OS V1R12.

One of the most common uses of IPSec is building virtual private networks (VPNs). Think of a VPN as a secure “tunnel” across a public network (like the Internet) that makes a remote link look and act like part of a private network. IPSec VPNs let companies send sensitive information safely over the Internet for internal communications or business-to-business links, and they also help protect sensitive traffic inside the company’s own network.

z/OS supports IKE and IPSec VPNs and includes these options and features:

Protocols: AH (Authentication Header) and ESP (Encapsulating Security Payload).

Encryption algorithms: ```Triple DES``` and ```AES``` (with multiple key lengths and modes).

Encapsulation modes: transport (protects payload only) and tunnel (protects the whole IP packet).

IKE negotiations: both IKEv1 and IKEv2; IKEv1 supports main and aggressive modes.

Authentication methods: pre-shared keys and digital signatures.

NAT traversal support for IPv4 networks.


| Layer       | Example Protocol | Typical Use             |
| ----------- | ---------------- | ----------------------- |
| Application | HTTPS (TLS)      | Web security            |
| Transport   | SSH              | Remote shell, tunneling |
| Network     | IPsec            | Secure VPNs             |
| Data        | AES, ChaCha20    | File encryption         |


## Key management

Key management means creating, distributing, storing, using, rotating and destroying cryptographic keys so encrypted data stays safe. Keys are the secret material that makes encryption work — if a key is lost or stolen, encryption fails.

Encryption strength comes from both the algorithm (```AES```, ```RSA```, ```ECC```) and the secrecy and quality of the keys. Weak, predictable, reused or exposed keys let attackers recover plaintext or forge signatures even if the algorithm is strong.

Public Key Infrastructure (PKI) uses certificates to link public keys to identities. Certificates are signed by trusted authorities (CAs), creating a trust chain. Ensures that a public key really belongs to the claimed entity.

Session keys are temporary symmetric keys used for encrypting a single session (fast, limited exposure). Ephemeral keys (from ```Diffie–Hellman``` or ```ECDHE```) are short-lived keys for secure key exchange, providing forward secrecy — even if long-term keys are compromised, past sessions remain safe.

Keys or certificates can be revoked if compromised.Trust chains are validated from a root CA down to the end certificate; any revoked certificate breaks the chain. Common revocation methods: CRLs (Certificate Revocation Lists) and OCSP (Online Certificate Status Protocol).

## Cryptography in Authentication and Authorization

**Hash-based Password Storage**
Instead of storing passwords in plain text, systems store hashes—one-way transformations of your password. To make it extra safe, we add a salt (random data) so identical passwords look different, and we apply key stretching (like bcrypt or Argon2) to slow down brute-force attacks. During login, the system hashes the entered password and compares it with the stored hash—never the password itself.

**Token-Based Systems**
After logging in, systems often give you a token instead of repeatedly asking for your password. JWTs (JSON Web Tokens) are common: they carry info like your user ID and expiration, and are signed so servers can trust them. Protocols like OAuth2 and OpenID Connect let third-party apps access your info securely without seeing your password.

**Digital Signatures in Access Control**
Digital signatures prove that code or firmware comes from a trusted source and hasn’t been tampered with. When software or updates are signed, your device can verify the signature before installing, ensuring security and integrity.

## Disk, Cloud and Secure Computation

**Disk and Database Encryption (BitLocker, LUKS)**
Whole-disk and volume encryption (e.g., BitLocker on Windows, LUKS on Linux) protect data at rest by encrypting the raw storage so files are unreadable without the correct key. They typically use strong symmetric ciphers and integrate with system boot / TPM to unlock automatically for legitimate users; databases can also use file- or tablespace-level encryption to protect backups and stolen drives.

**Cloud Data Security (Client-Side Encryption)**
Encrypting data before it leaves the client (client-side encryption) ensures the cloud provider never sees plaintext — you hold the keys. This protects against provider compromise or insider access but shifts responsibility to secure key storage, key rotation, and sharing policies (how to let collaborators decrypt).

**Homomorphic Encryption and Secure Computation**
Homomorphic encryption lets computations run directly on encrypted data so a server can process inputs without seeing them; results decrypt to the same outcome as if processed in plaintext. Related approaches — secure multi-party computation (MPC) and trusted execution environments (TEEs) — trade off performance, trust model, and complexity; these are promising for privacy-preserving analytics but currently more costly than standard encryption.

## Cryptography in Distributed Systems

**Blockchain**
Blocks are chained by cryptographic hashes so any change breaks the chain — this provides tamper-evidence. Merkle trees let a node verify a single transaction’s inclusion efficiently (verify a short hash path instead of the whole block).

**Consensus Mechanisms**
Proof-of-Work (PoW) requires solving costly puzzles (hashing until a target is found) to make rewriting history expensive — Bitcoin is the canonical PoW example. Proof-of-Stake (PoS) selects validators based on stake and penalties, reducing energy cost (Ethereum migrated to PoS). Byzantine fault-tolerant and hybrid schemes aim to tolerate malicious or faulty nodes with different performance/trust trade-offs.

**Wallets and Addresses**
Users’ identities on blockchains are public/private key pairs; addresses are derived from public keys so signing proves ownership. Common signature schemes include ```ECDSA``` and ```EdDSA (Ed25519)```, chosen for compact keys and fast verification.

**Smart Contracts**
Smart contracts are programs whose execution and state changes are cryptographically enforced by the blockchain (e.g., EVM running Solidity code). Security pitfalls include reentrancy, integer overflow/underflow, and access-control bugs — these have real monetary consequences, so audits and formal checks matter.

## Applied Cryptography in Modern Systems

**Secure Messaging**
Modern secure messengers use layered cryptography: X3DH (extended Triple Diffie–Hellman) establishes initial shared secrets between parties, then the Double Ratchet (a DH ratchet + symmetric-key ratchet) advances keys for each message so that compromise of one key doesn’t expose everything. This design gives forward secrecy (past messages stay safe if keys are later leaked) and post-compromise security (after a compromise, security can be restored automatically as new messages/keys are exchanged).

**Zero-Knowledge Proofs**
A zero-knowledge proof lets you prove you know a secret (or that a statement is true) without revealing the secret itself. zk-SNARKs are a popular class of succinct, non-interactive proofs used in privacy-focused systems (e.g., Zcash) to prove transaction validity without revealing amounts or addresses. ZK tools are also being applied to identity, verifiable computation, and privacy-preserving audits.

**Multi-Party Computation (MPC)**
MPC lets multiple parties jointly compute a function over their private inputs without revealing those inputs to each other. Practical uses include private auctions, joint analytics on sensitive datasets, and privacy-preserving machine learning. MPC protocols vary by performance and trust model — some need many rounds and heavy crypto, others use precomputation or rely on hybrid trust (e.g., a small trusted setup) to be faster.

## See also

https://hackyourmom.com/pryvatnist/pgp/
