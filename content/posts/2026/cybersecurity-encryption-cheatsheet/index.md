---
date: '2026-09-04T11:00:00+03:00'
draft: false
title: 'Cybersecurity and Encryption Cheatsheet: Algorithms, Protocols and Attacks'
tags: ["cheatsheet", "cryptography", "security"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Symmetric and asymmetric ciphers, hashes, TLS, authentication and access control, key management, and the attacks they exist to stop."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

A working map of applied cryptography and the security machinery built on top of it. Plenty of the algorithms below are here because you will meet them in old code and old exam questions, not because you should deploy them — so each one comes with its current status, which is the part that reference tables usually get wrong.

## Symmetric Encryption

Symmetric algorithms use the *same* secret key to encrypt and decrypt. They are fast, which is why they carry the actual data in nearly every protocol, and their weak point is key distribution: both parties need the key, and getting it to them safely is a separate problem — the one asymmetric cryptography solves.

| Algorithm | Type | Block size | Key size | Status |
| --- | --- | --- | --- | --- |
| AES | Block | 128 bits | 128 / 192 / 256 bits | **Current standard** |
| DES | Block | 64 bits | 56 bits | Broken — brute-forceable |
| 3DES | Block | 64 bits | 112 / 168 bits | Disallowed since 2023 |
| Blowfish | Block | 64 bits | up to 448 bits | Superseded |
| Twofish | Block | 128 bits | up to 256 bits | Unbroken, little used |
| Serpent | Block | 128 bits | up to 256 bits | Unbroken, little used |
| MARS | Block | 128 bits | 128–448 bits | Unbroken, little used |
| IDEA | Block | 64 bits | 128 bits | Superseded |
| CAST-128 | Block | 64 bits | up to 128 bits | Superseded |
| RC2 | Block | 64 bits | up to 1024 bits | Insecure |
| RC5 | Block | variable | up to 2040 bits | Rarely used |
| RC6 | Block | 128 bits | up to 2040 bits | Rarely used |
| RC4 | Stream | — | up to 2048 bits | **Prohibited** |

**AES** is the one that matters. It is the current standard, it is implemented in hardware on every modern CPU, and unless you have a specific reason to do otherwise it is the answer. Note that the key size varies but the block size does not: AES-256 still processes 128-bit blocks.

**DES** fell to brute force decades ago — 56 bits is simply not enough key material. **3DES** was the stopgap, applying DES three times with two or three distinct keys. It is slow, and NIST deprecated it through 2023 and disallowed it for encryption after 31 December 2023 in SP 800-131A Rev. 2; decryption of legacy data is still permitted. If you find 3DES protecting anything live, that is a finding.

The 64-bit block ciphers deserve a note of their own. **Blowfish**, **3DES**, **IDEA**, **CAST-128** and **RC2** all use 64-bit blocks, and that alone is now a weakness: the Sweet32 birthday attack recovers plaintext from long-lived connections encrypted with any 64-bit block cipher, without touching the key. Blowfish is fast and was widely trusted for years, but its own designer has recommended moving to AES or Twofish for a long time.

**Twofish**, **Serpent** and **MARS** were AES finalists alongside Rijndael, which won. None of them is broken; they simply lost, and losing meant no hardware acceleration and no ecosystem. MARS was IBM's entry — the name is not an acronym, whatever expansions you may see attached to it.

**RC4** is the important negative. It was everywhere, it was fast, and it is now formally prohibited in TLS by RFC 7465. It must not be negotiated in any TLS version.

One clarification worth making because reference cards routinely file it in the wrong drawer: **Kerberos is not an encryption algorithm.** It is a network authentication protocol that *uses* symmetric cryptography to protect its messages. More on it below.

## Asymmetric Encryption

Asymmetric cryptography — public-key cryptography — uses two mathematically related keys. The public key can be handed to anyone and is used to encrypt; the private key stays secret and is used to decrypt. Run the pair the other way round and you get digital signatures, which is where the **non-repudiation** property comes from: only the holder of the private key could have produced the signature, so they cannot credibly deny it.

The trade-off against symmetric crypto is speed. Asymmetric operations are orders of magnitude slower, so in practice they are used to authenticate the parties and agree on a symmetric key, and the symmetric cipher does the bulk work. That hybrid arrangement is what TLS is.

- **RSA** — the workhorse for key transport, signatures and encryption at rest, named for Rivest, Shamir and Adleman. Still fine at 2048-bit keys and above; 1024-bit RSA is retired.
- **Diffie-Hellman** — a key *exchange* method, not an encryption algorithm. It lets two parties derive a shared secret over a public channel without ever transmitting it. Ephemeral variants (DHE, ECDHE) are what give modern TLS its forward secrecy.
- **ECC (Elliptic Curve Cryptography)** — the same operations built on elliptic curves, which gives equivalent security at much smaller key sizes. A 256-bit ECC key is roughly comparable to a 3072-bit RSA key, which is why ECC dominates on constrained devices and modern TLS alike.
- **DSA** — the Digital Signature Algorithm, for signatures only. ECDSA is its elliptic-curve version and the one you will actually meet; Ed25519 has largely displaced both in new systems.
- **ElGamal** — a public-key system usable for both signatures and encryption. Mostly of academic and historical interest, though it survives inside parts of OpenPGP.
- **PGP** — not an algorithm but a tool, combining symmetric and asymmetric encryption for secure storage and messaging. See the [GPG post](../../2025/gpg/) for the practical side of using it.

### Post-quantum

Every algorithm in this section rests on problems a sufficiently large quantum computer would solve efficiently. NIST published the first post-quantum standards in August 2024: **ML-KEM** (FIPS 203) for key encapsulation, **ML-DSA** (FIPS 204) and **SLH-DSA** (FIPS 205) for signatures. Hybrid key exchange combining ECDHE with ML-KEM is already deployed in mainstream browsers and TLS libraries. Symmetric cryptography and hashing are much less affected — the practical response there is larger keys and longer digests, not new algorithms.

## Hash Functions

A hash takes input of any length and produces a fixed-length digest. It is one-way: you cannot recover the input, and finding two inputs with the same digest — a collision — should be computationally infeasible. Hashes underpin signatures, integrity checks, and password storage.

| Family | Digest size | Status |
| --- | --- | --- |
| SHA-2 (SHA-256, SHA-512) | 224–512 bits | **Current standard** |
| SHA-3 (Keccak) | 224–512 bits | Current, different internal design |
| BLAKE2 / BLAKE3 | variable | Current, very fast |
| SHA-1 | 160 bits | Broken — collisions demonstrated |
| RIPEMD-160 | 160 bits | Unbroken but niche |
| Whirlpool | 512 bits | Unbroken but niche |
| Tiger | 192 bits | Legacy |
| MD5 | 128 bits | Broken |
| MD4, MD2 | 128 bits | Broken |

**SHA-2** is the default choice, with SHA-256 the most common member. **SHA-3** is not a patch on SHA-2 but a structurally different design chosen through open competition, kept in reserve in case SHA-2 ever falls. **BLAKE2** and its successor BLAKE3 are faster than both and appear widely outside standards-mandated contexts.

**SHA-1** is finished. A practical collision was demonstrated in 2017 and chosen-prefix collisions followed in 2020. NIST formally retired it in 2022 and requires it gone by the end of 2030. **MD5** has been collision-broken since 2004 and is unfit for any security purpose — its only defensible remaining use is as a non-adversarial checksum.

The whole MD family produces 128-bit digests: MD2, MD4 and MD5 alike. All are broken.

One deliberate omission: **none of these belongs anywhere near a password database.** General-purpose hashes are designed to be fast, which is precisely wrong for passwords. Use a deliberately slow, memory-hard function — Argon2id, scrypt, or bcrypt.

## TLS, Not SSL

SSL is dead. Every version of it — SSL 2.0 and SSL 3.0 — is formally deprecated, SSL 3.0 by RFC 7568 in 2015. Its successor is TLS, and TLS 1.0 and 1.1 are deprecated too, by RFC 8996 in 2021. Only **TLS 1.2 and TLS 1.3** should be enabled anywhere. The word "SSL" survives in library names, configuration keys and job titles, but the protocol underneath is TLS.

TLS has the same two-part structure SSL had: a **handshake** that authenticates the server and establishes a shared secret, and a **record protocol** that carries application data encrypted under that secret.

The classic handshake runs roughly like this:

1. The client sends a hello listing the protocol versions and cipher suites it supports.
2. The server replies with the version and cipher suite it has chosen.
3. The server presents its certificate to prove its identity.
4. The client validates that certificate against its trust store, and the two sides complete the key exchange.
5. Everything after that is encrypted with the agreed symmetric key.

**TLS 1.3 compresses this considerably.** It completes in one round trip instead of two, encrypts the certificate rather than sending it in the clear, and strips out every cipher suite with a known weakness — no RC4, no 3DES, no static RSA key exchange, no renegotiation. Its suites are AES-GCM, AES-CCM and ChaCha20-Poly1305, all of them authenticated encryption. Forward secrecy is mandatory, meaning a stolen server key cannot decrypt yesterday's captured traffic.

If you are configuring a server today: TLS 1.2 and 1.3 only, ECDHE key exchange, AEAD ciphers, HSTS enabled. Mozilla's SSL Configuration Generator produces sound configs for most web servers.

## Authentication

Authentication answers "who are you?" and is conventionally broken into three factors — something you know, something you have, something you are.

| Method | Factor | Notes |
| --- | --- | --- |
| Username and password | Know | Weakest alone; the baseline everywhere |
| Two-factor / MFA | Know + have | Password plus a code from an app or token |
| Biometric | Are | Fingerprint, face; convenient, not revocable |
| Smart card | Have | Physical card with a chip, read by a terminal |
| Token-based | Have | USB key or hardware token holding credentials |
| Certificate-based | Have | Digital certificate issued by a trusted authority |

Multi-factor is the single highest-value control in this table, but not all second factors are equal: SMS codes are interceptable through SIM swapping and network attacks, app-generated TOTP codes are phishable in real time, and only hardware security keys using WebAuthn/FIDO2 resist phishing outright, because the key checks the origin before it will sign anything.

The biometric caveat is worth stating plainly: you cannot reissue a fingerprint. Biometrics work well as a local unlock gesture on a device that holds a real credential, and poorly as a secret transmitted over a network.

### Authentication protocols

- **Kerberos** — a network authentication protocol using symmetric cryptography and a trusted third party, the Key Distribution Center. It issues time-limited tickets rather than passing passwords around, which protects against replay, eavesdropping and machine-in-the-middle attacks. It is the backbone of Active Directory authentication.
- **OAuth 2.0** — an *authorisation* framework, not authentication. It lets a user grant an application access to their data without handing over their password. **OpenID Connect** is the identity layer built on top of it, and that is the piece that actually does authentication — a distinction that gets blurred constantly and matters when you are designing a login flow.
- **SAML** — Security Assertion Markup Language, an XML standard for exchanging authentication and authorisation data between an identity provider and a service provider. Older than OIDC and still dominant in enterprise single sign-on.
- **RADIUS** — Remote Authentication Dial-In User Service (RFC 2865), providing centralised authentication, authorisation and accounting for network access. Its name is a fossil from the dial-up era; it is now what sits behind enterprise Wi-Fi and VPN logins. It encrypts only the password field, not the whole exchange.
- **TACACS+** — Terminal Access Controller Access-Control System Plus (RFC 8907), Cisco's alternative for device administration. It separates authentication, authorisation and accounting into independent exchanges and encrypts the entire payload, which is its main advantage over RADIUS. Plain TACACS and XTACACS are obsolete.

## Access Control Models

Once you know who someone is, you decide what they may do.

- **MAC (Mandatory Access Control)** — a central authority sets the rules and users cannot override them. Labels and clearances; standard in government and military systems. SELinux and AppArmor are the everyday Linux examples.
- **DAC (Discretionary Access Control)** — the owner of a resource decides who gets access. Unix file permissions are DAC, and so is a shared folder on a home NAS.
- **RBAC (Role-Based Access Control)** — permissions attach to roles, users are assigned roles. The dominant model in large organisations because it scales with headcount instead of with individuals.
- **ABAC (Attribute-Based Access Control)** — decisions evaluate attributes of the user, resource, action and context, so you can express rules like "finance staff, on a managed device, during business hours." More expressive than RBAC and correspondingly harder to reason about.

## Remote Access

- **VPN** — an encrypted tunnel over the public internet giving remote access to a network. **Site-to-site** VPNs join two networks permanently; **client** VPNs connect individual devices.
- **IPsec** — a protocol suite securing traffic at the IP layer, and the basis of most site-to-site VPNs.
- **SSH** — secure remote shell access over an untrusted network, and also the transport for `scp` and `sftp`.
- **RDP** — Remote Desktop Protocol, for graphical remote access to Windows machines. Never expose it directly to the internet; put it behind a VPN or a gateway.
- **PPTP** — obsolete and insecure. Its MS-CHAPv2 authentication was broken in 2012 and the whole exchange can be reduced to a single DES key. Do not use it; modern equivalents are WireGuard, OpenVPN or IKEv2/IPsec.
- **802.11** — the IEEE standard family for *wireless* LANs. Use WPA3, or WPA2 with AES-CCMP at minimum; WEP and TKIP are broken.
- **RATs** — remote access trojans, the attacker's version of all of the above: malware granting unauthorised remote control of a victim machine.

## Key Management and the Certificate Life Cycle

Cryptography fails at key management far more often than at the mathematics. A certificate moves through a predictable life cycle:

1. **Key generation** — the requesting entity generates a key pair.
2. **Identity submission** — it presents its identity to a Certificate Authority.
3. **Registration** — the CA verifies that identity.
4. **Certification** — the CA signs a certificate binding the entity's public key to its identity, using the CA's own private key.
5. **Distribution** — the certificate is issued to the entity and made publicly available.
6. **Usage** — the entity authenticates itself and establishes secure communications.
7. **Expiration or revocation** — every certificate has a time limit, and a compromised one is revoked before it.
8. **Renewal** — a fresh key pair and certificate are issued as needed.
9. **Recovery** — a defined procedure for a lost private key.
10. **Archival** — certificates and keys are retained securely for audit.

An **HSM (Hardware Security Module)** is a dedicated physical device that generates keys, signs and verifies, and encrypts and decrypts, with the private key never leaving the hardware. Cloud providers offer managed equivalents, and the design goal is the same: make key extraction a physical problem rather than a software one.

Two practical notes the life cycle above does not capture. Public TLS certificates have short lifetimes now — a little over a year and shrinking — which makes automated renewal mandatory rather than optional; ACME and Let's Encrypt exist for exactly this. And revocation is the weak link: CRLs and OCSP both have well-known reliability problems, which is a large part of why the industry moved to short-lived certificates instead.

### Key management vs PKI management

The two terms get used interchangeably and are not the same thing.

| Key management | PKI management |
| --- | --- |
| Secure creation, distribution, archival and deletion of cryptographic keys | Production, distribution and maintenance of digital certificates and their keys |
| Keys encrypt and decrypt messages, authenticate users, and establish secure connections | Certificates confirm the identity of parties to a transaction and guarantee data integrity |
| Ensures keys are used correctly and protected from unauthorised access or misuse | Establishes trust between parties, and revokes certificates that are compromised or invalid |

The short version: key management is about protecting secrets; PKI management is about establishing who those secrets belong to.

## Attacks

### Spoofing

Spoofing means faking an identity to gain trust.

| Type | How it works | Defence |
| --- | --- | --- |
| IP spoofing | Sending packets from a forged source address | Ingress filtering (BCP 38) |
| Email spoofing | Forging the `From` address to launch phishing or deliver malware | SPF, DKIM, DMARC |
| DNS spoofing | Poisoning DNS records to redirect traffic to an attacker's server | DNSSEC, DNS over HTTPS |
| Caller ID spoofing | Faking a phone number to impersonate a trusted caller | STIR/SHAKEN |
| MAC spoofing | Changing a device's MAC address to impersonate another host | 802.1X port authentication |

### Machine-in-the-middle

The attacker sits between two parties, reading or altering traffic that both sides believe is private.

- **Wi-Fi hijacking** — a rogue access point, often a purpose-built device like a Wi-Fi Pineapple, that clients associate with instead of the real network.
- **SSL stripping** — downgrading an HTTPS connection to HTTP so the attacker can read it. HSTS is the defence: it tells browsers never to use plaintext for that domain.
- **Email hijacking** — intercepting and altering mail between two parties. The commercial version is business email compromise, where invoice details are quietly changed in transit.
- **Banking trojans** — malware on the endpoint itself, watching sessions with a bank to harvest credentials and authorise fraudulent transfers.

Certificate validation is what stops most of these at the protocol level, which is why "just click through the certificate warning" is worse advice than it sounds.

### Denial of service

DoS attacks exhaust a resource — bandwidth, connection state, CPU — until the service stops answering legitimate requests. Distributed versions (DDoS) source the traffic from many hosts at once.

| Attack | Mechanism |
| --- | --- |
| Ping flood | Floods the target with ICMP echo requests |
| SYN flood | Opens half-finished TCP connections until the connection table is exhausted |
| UDP flood | Saturates the network with UDP packets |
| HTTP flood | Sends large volumes of legitimate-looking HTTP requests |
| DNS flood | Overwhelms the target's DNS servers |
| Smurf attack | Sends ICMP echoes to broadcast addresses so replies flood the victim |
| NTP amplification | Abuses open NTP servers to multiply traffic volume at the target |
| Slowloris | Opens many connections and sends partial HTTP requests slowly, holding sockets open |

The distinction that matters operationally is between **volumetric** attacks, which try to fill the pipe, and **resource-exhaustion** attacks like Slowloris, which use very little bandwidth and instead consume connection slots. The first is absorbed with capacity and upstream scrubbing; the second is defeated with connection limits and timeouts, and no amount of bandwidth helps.

Amplification attacks — NTP, DNS, memcached and others — work by sending a small spoofed request to a server that answers with a much larger response directed at the victim. They depend on both IP spoofing and misconfigured public servers, which is why closing open resolvers and deploying ingress filtering helps everyone rather than just the operator who does it. The Smurf attack is largely historical for the same reason: routers stopped forwarding directed broadcasts by default.
