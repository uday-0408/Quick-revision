# INS Exam Study Guide — Units 8, 9, 10
### Built from your L.J. Institute Practice Book (SEM-VI-2026)

---

## How To Use This Tonight

1. Read **Part 0 (Prerequisites)** once, slowly. Everything else builds on it.
2. Go unit by unit. In each unit: read the "Topics" list, skim the concept, then go through MCQs and theory answers.
3. For theory questions, don't memorize word-for-word — memorize the **structure** (the bullet headings). In the exam, write those headings and expand each in 1–2 lines. That gets you most of the marks even under time pressure.
4. The ElGamal numericals (bonus section at the end) are pure method — once you know the 4 formulas, you can solve *any* version of that question, not just these four.

---

## PART 0 — Prerequisites (read this first)

**Symmetric-key crypto**: same key encrypts and decrypts (AES, DES, IDEA). Fast, but the key must be shared secretly beforehand — that's the "key distribution problem" Unit 8 is about.

**Asymmetric (public-key) crypto**: two mathematically linked keys — a **public key** (shared with everyone) and a **private key** (kept secret). Data encrypted with one can only be decrypted with the other. Used for key exchange, digital signatures, and certificates.

**Hash function**: takes any input, produces a fixed-size fingerprint (digest). Used to check integrity — if the data changes even slightly, the hash changes completely. Hashes are one-way (can't reverse them).

**Digital signature**: sender hashes the message, then encrypts the hash with their **private key**. Anyone can decrypt it with the sender's **public key** and compare it to a fresh hash of the message — this proves *authenticity* (came from sender) and *integrity* (unaltered).

**Why "certificates" exist**: public keys are useless if you can't be sure *whose* key it really is (a fake key could be swapped in — man-in-the-middle). A **digital certificate** is a trusted third party (Certificate Authority) vouching "this public key belongs to this identity," signed with the CA's own private key.

**Modular arithmetic basics** (needed for ElGamal numericals):
- `a mod n` = remainder when a is divided by n.
- Negative numbers: keep adding `n` until positive. E.g., `-34 mod 18`: -34+36 = **2**.
- **Modular inverse** of K mod n: the number X such that `(K × X) mod n = 1`. Find by trial: multiply K by 1,2,3... until you hit a number ≡1 mod n.

**OSI Layers** (needed for SSL/IPsec/VLAN questions): Application(7) → Presentation(6) → Session(5) → **Transport(4)** → **Network(3)** → **Data Link(2)** → Physical(1).
- SSL sits between Transport and Application.
- IPsec operates at the Network layer.
- VLAN operates at the Data Link layer.

---

# PART 1 — UNIT 8: Key Management, X.509, PKI, Kerberos

## Topics To Study
- The 4 methods of public key distribution (announcement, directory, authority, certificates)
- X.509 certificate: what fields it has, which version added what
- PKI: what it is, its components (CA, RA, certificate database), why certificates get revoked, CRL
- Kerberos: what it is, why it exists, its servers (AS, TGS), port number, origin (MIT)

## Prerequisite Concept for This Unit
The **core problem** all of Unit 8 solves: *"I have your public key — but how do I know it's really yours, and not an attacker's?"* Every topic here (certificates, PKI, Kerberos) is a different solution to that trust problem — certificates use a signed document; PKI is the whole system that issues/manages those documents; Kerberos solves it differently, using trusted tickets instead of public keys at all.

---

## MCQ Solutions — Grouped by Topic

### Group A: Public Key Distribution (the 4 schemes)
There are exactly **4 categories**: (1) Public announcement, (2) Publicly available directory, (3) Public-key authority, (4) Public-key certificates. Increasing security in roughly that order.

| Q# | Question | Ans | Why |
|---|---|---|---|
| 269 | Which is NOT a public-key distribution means? | **B: Hashing Certificates** | Not one of the 4 real categories — a distractor term. |
| 270 | Most secure distribution system | **A: Public-Key Certificates** | Signed by a trusted CA, no need to contact anyone live — most tamper-resistant. |
| 271 | Which systems use a timestamp? | **C: i) and iv)** | Public-Key Certificates *and* Public-Key Authority both use timestamps (certs for expiry, authority for freshness of each request). |
| 272 | Timestamp used as an *expiration date* | **A: Public-Key Certificates** | Certificates literally carry a "valid until" date — that's their timestamp's role. |
| 278 | Publicly available directory more secure than | **B: Public announcements** | Directory is maintained/authenticated centrally; announcement is just "shout your key to everyone" — anyone can spoof it. |
| 287 | USENET relates to which scheme | **B: Public announcements** | Broadcasting your key to a public forum with no verification = public announcement. |

### Group B: X.509 Certificates
Know the version history: **v1** = basic fields. **v2** added Issuer & Subject **Unique Identifiers**. **v3** added **Extensions**.

| Q# | Question | Ans | Why |
|---|---|---|---|
| 267 | Subject unique identifier added in which version? | **B: 2** | v2's headline addition. |
| 292 | Issuer unique identifier added in which version? | **B: 2** | Same update as above — both unique IDs came in v2. |
| 275 | Extensions added in which version? | **C: 3** | v3's headline addition — lets certs carry extra custom fields. |
| 268 | NOT a field of X.509 | **B: "Serial Modifier"** | Real fields: Serial Number, Issuer Name, Validity, Subject Name, Public Key, Signature — "Serial Modifier" doesn't exist. |
| 273 | X.509 recommends which algorithm? | **A: RSA** | RSA is the textbook-recommended algorithm for X.509 key pairs. |
| 281 | Standard defining certificate structure/fields | **A: X.509** | This is the literal definition of X.509. |
| 286 | PKI standard identifying format of public key certificates | **B: X.509** | Same idea — X.509 = the certificate format standard. |
| 279 | MongoDB supports ___ certificate authentication | **B: x.509** | Just recall the real standard name (the other options are typos). |

**Quick sketch of X.509 fields (for Q301's "draw the format")**:
```
 ┌───────────────────────────────┐
 │ Version                       │
 │ Serial Number                 │
 │ Signature Algorithm ID        │
 │ Issuer Name                   │
 │ Validity Period (Not Before/  │
 │                   Not After)  │
 │ Subject Name                  │
 │ Subject Public Key Info       │
 │ Issuer Unique ID   (v2+)      │
 │ Subject Unique ID  (v2+)      │
 │ Extensions         (v3+)      │
 │ Signature (by CA's priv key)  │
 └───────────────────────────────┘
```

### Group C: PKI (Public Key Infrastructure)
PKI = the entire *system* around certificates: who issues them, who verifies identity, where they're stored, how they're revoked.

| Q# | Question | Ans | Why |
|---|---|---|---|
| 266 | True statement about PKI | **A** | PKI = digital certificates + public-key crypto + CAs, giving enterprise-wide security — the textbook definition. |
| 274 | Functions in PKIX architectural model | **D: 7** | Memorize the number: 7 functions (registration, initialization, certification, key pair recovery, key pair update, revocation request, cross-certification). |
| 276 | Why revoke a certificate before expiry | **D: All of the mentioned** | Any of: user no longer certified, CA compromised, or user's private key compromised — all are valid revocation reasons. |
| 277 | CRL stands for | **C: Certificate Revocation List** | The published list of revoked (not-yet-expired) certificates. |
| 282 | Key management practice requires | **A** | Keys must be truly random, using the full keyspace — weak/predictable keys defeat the whole system. |
| 288 | Elements of PKI | **D: All of them** | CA, RA, and certificate database are all core PKI components. |
| 293 | Name of the issuer of PKI | **A: Certificate authorities** | The CA is literally the entity that *issues* certificates. |
| 299 (theory) | see Theory section below | | |

### Group D: Kerberos
Kerberos = a **ticket-based** authentication system (not certificate-based) using a trusted third party.

| Q# | Question | Ans | Why |
|---|---|---|---|
| 283 | Kerberos port | **B: 88** | Fixed, memorize it — port 88. |
| 284 | Kerberos developed at | **A: MIT** | Project Athena, MIT. |
| 285 | Kerberos takes into account | **D: All of them** | Client, KDC (Key Distribution Center), and TGS (Ticket Granting Server) are all part of the model. |
| 295 | Factor to consider when implementing Kerberos | **A** | Kerberos needs a centrally managed database of all user/resource passwords — that's a real operational burden/risk. |
| 280 | Client requests from KDC a ___ for access | **A: ticket** | This is Kerberos's whole mechanism — tickets, not keys, grant access. |

### Group E: General / Misc
| Q# | Question | Ans | Why |
|---|---|---|---|
| 289 | Example of an authentication token | **B: Smart card** | A physical/possession-based factor — Pin is a knowledge factor, not a token. |
| 290 | Virus spreads through | **D: All of these** | Floppy, CD, email attachments — all are valid infection vectors. |
| 291 | For confidentiality, data to be sent is | **C: Encrypted** | Confidentiality = encryption, by definition. |
| 294 | Public key enc/dec not preferred because | **D: All of the mentioned** | It's slow, hardware/software intensive, and has high computational load — all true, which is why hybrid systems (symmetric for bulk data) are used instead. |

---

## Theory Solutions (296–310)

**Q296/297 — Categories of public key distribution (list + elaborate)**
The 4 categories:
1. **Public announcement** — broadcast your public key openly (e.g., attach to emails). *Weakness*: anyone can forge a key and claim it's yours.
2. **Publicly available directory** — a maintained, (ideally authenticated) directory that maps name→public key. *Better*, but the directory itself is a single point of attack.
3. **Public-key authority** — a live, trusted central server that clients query in real time for someone's current public key, using timestamps and challenge-response to prevent replay. *More secure*, but requires the authority to be online for every exchange.
4. **Public-key certificates** — the authority pre-signs a certificate binding identity to public key. Users exchange certificates directly without contacting the authority each time. *Most practical and most secure.*

**Q298 — Key distribution + public-key authority & certificate methods**
Key distribution = the problem of getting public keys to the right people securely, without impersonation.
- *Public-key authority*: A trusted central server holds everyone's keys. When A wants B's key, A sends a timestamped request to the authority, which replies with B's key signed by the authority's private key (so A can verify it's authentic and fresh).
- *Public-key certificate*: The authority (CA) issues A a certificate = {A's identity + A's public key} signed with the CA's private key. A can now hand this certificate directly to anyone (like B), who verifies it using the CA's well-known public key — no live contact with the CA needed.

**Q299 — Components of PKI (5 marks)**
- **Certificate Authority (CA)**: issues, signs, and revokes digital certificates; the root of trust.
- **Registration Authority (RA)**: verifies user identity before the CA issues a certificate (offloads identity-checking from the CA).
- **Certificate Database/Repository**: stores issued certificates and their status.
- **Certificate Revocation List (CRL)**: published list of revoked certificates.
- **PKI Clients/End entities**: users, devices, or applications that hold and use certificates.
- **Key Recovery/Archival systems**: for recovering encrypted data if a private key is lost.

**Q300 — Discuss X.509 certificates (5 marks)**
- Defines the *standard format* for public key certificates.
- Fields: Version, Serial Number, Signature Algorithm, Issuer, Validity Period, Subject, Subject's Public Key, (v2) Issuer/Subject Unique ID, (v3) Extensions, CA's digital Signature.
- Issued and signed by a CA — the signature is what lets anyone verify authenticity without contacting the CA live.
- Used everywhere: HTTPS/TLS, email (S/MIME), code signing.

**Q301 — X.509 format diagram + reasons to revoke early**
- Diagram: see the box diagram above.
- Reasons to revoke before expiry: (a) the CA's own certificate is suspected compromised, (b) the user's private key is suspected compromised/stolen, (c) the user is no longer authorized/certified by this CA (e.g., left the organization).

**Q302 — Why is PKI used? (3 marks)**
- Solves the trust problem in public-key crypto: proves a public key genuinely belongs to a claimed identity.
- Enables secure communication, authentication, and non-repudiation at scale (enterprise/internet-wide), without every pair of users having to meet in person to exchange keys.
- Backbone of HTTPS, secure email, code signing, VPNs.

**Q303 — How PKI works; CA & RA**
- Flow: User generates key pair → RA verifies identity → CA issues & signs certificate → Certificate published/distributed → Others verify signature using CA's public key → Certificate can later be revoked (added to CRL) if compromised.
- **CA (Certificate Authority)**: the trusted entity that actually signs and issues certificates; the root of trust in the system.
- **RA (Registration Authority)**: acts as the verifier/gatekeeper — checks the applicant's identity/credentials *before* forwarding the request to the CA. (CA trusts, RA verifies.)

**Q304/305 — Explain Kerberos in detail; how it authenticates users (5 marks)**
- Kerberos is a **ticket-based** network authentication protocol (not certificate-based), built around a trusted **Key Distribution Center (KDC)**, which has two parts:
  - **Authentication Server (AS)**: verifies the user's identity at login, issues a **Ticket-Granting Ticket (TGT)**.
  - **Ticket-Granting Server (TGS)**: using the TGT, issues **service tickets** for specific application servers.
- Flow: Client logs in → AS verifies credentials, returns TGT (encrypted) → Client presents TGT to TGS to request access to a service → TGS issues a service ticket → Client presents service ticket to the Application Server → Server grants access.
- Key benefit: the user's password is never sent over the network repeatedly, and tickets are time-limited (reducing replay risk).

**Q306 — Kerberos across administrative domains (cross-realm, 3 marks)**
- Each administrative domain (realm) has its own KDC.
- For cross-realm auth: the client's local KDC and the remote realm's KDC must have a trust relationship (inter-realm key).
```
 [Client] → [Local AS] → [Local TGS] 
     → (presents ticket to) → [Remote TGS in other realm]
     → [Remote Application/HTTP Server]
```
- The local TGS issues a special "referral" ticket usable at the remote realm's TGS, which then issues the actual service ticket for the remote server.

**Q307 — Requirements of Kerberos; roles of servers (3 marks)**
- **Requirements**: Secure (no password sent in clear), Reliable (available, distributed for redundancy), Transparent (user shouldn't notice the process), Scalable (support many users/servers).
- **Servers**: 
  - AS — authenticates the user at login, issues TGT.
  - TGS — issues service tickets using the TGT.
  - Application Server — the actual service the user wants to access, validates the service ticket.

**Q308 — Kerberos + HTTP server diagram (3 marks)**
```
 [You] → login → [AS] → returns TGT
 [You] → TGT → [TGS] → returns Service Ticket (for HTTP server)
 [You] → Service Ticket → [HTTP Server] → grants access
```
(Substitute your own name for "You" as the question asks.)

**Q309 — Public key cryptography vs PKI (2 marks)**
- **Public key cryptography** = the mathematical technique (algorithms like RSA/ElGamal) using key pairs for encryption/signatures.
- **PKI** = the entire *management system* around it — issuing, verifying, storing, and revoking the certificates that make public key crypto trustworthy at scale. (Crypto is the tool; PKI is the trust infrastructure around the tool.)

**Q310 — Public announcement & public key directory methods (2 marks)**
(Same as covered in Q296/297 above — public announcement = open broadcast, no verification; directory = maintained, more trustworthy repository mapping identities to keys.)

---

# PART 2 — UNIT 9: PGP, SSL, IPsec

## Topics To Study
- PGP: purpose, algorithms used, key components, keyservers, revocation
- SSL: layer it operates at, the 4 protocols within SSL (Handshake, Change Cipher Spec, Alert, Record), handshake steps, alert types
- IPsec: AH vs ESP, Tunnel vs Transport mode, IKE

## Prerequisite Concept for This Unit
This unit is about **applying** the Unit 8 concepts (keys, certificates, signatures) to real protocols that secure actual traffic: **PGP** secures *email*, **SSL** secures a *TCP connection* (e.g., HTTPS), **IPsec** secures *IP packets themselves* (network layer, protocol-independent). Same building blocks (hashing, symmetric+asymmetric encryption, certificates), different layer/use-case.

---

## MCQ Solutions — Grouped by Topic

### Group A: PGP (Pretty Good Privacy)
| Q# | Question | Ans | Why |
|---|---|---|---|
| 311 | Creator of PGP | **A: Phil Zimmermann** | Created it in 1991. |
| 312 | PGP primarily used for | **B: Email Encryption** | Its original and main use case. |
| 313 | Crypto algorithms commonly used in PGP | **A: RSA and AES** | RSA for key exchange/signing, symmetric cipher (book states AES) for bulk message encryption. *(Note: classic PGP originally used IDEA; modern OpenPGP supports AES too — go with the book's answer for the exam.)* |
| 314 | Purpose of a "revocation certificate" | **C: To revoke a compromised or lost private key** | Lets you announce "don't trust my old key anymore" even without the private key. |
| 315 | PGP key pair component kept secret | **B: Private Key** | Public key is shared; private key never leaves the owner. |
| 316 | Purpose of "Keyserver" | **B: To distribute public keys** | A directory service where people publish/look up public keys. |
| 317 | PGP version that introduced OpenPGP standard support | **B: PGP 5.0** | PGP 5.0's format became the basis for the OpenPGP standard (RFC 2440). |
| 318 | Key used for encrypting the message in PGP | **C: Recipient's Public Key** | Only the recipient's matching private key can then decrypt it. |
| 319 | How PGP ensures confidentiality | **A: By using symmetric-key encryption** | The message itself is encrypted with a fast one-time symmetric session key; that session key is then separately wrapped in the recipient's public key (hybrid encryption). |
| 320 | Algorithm commonly used to generate PGP key pairs | **C: RSA** | Standard PGP key-pair generation algorithm. |

**PGP workflow diagram (for Q321/322/325/326)**:
```
Sender side:
 Message → [Hash] → [Sign with sender's private key] → Signature
 Message + Signature → [Compress]
 → [Encrypt with random session key (symmetric)]
 → [Encrypt session key with recipient's public key]
 → [Radix-64 encode for email compatibility] → Send

Receiver side: reverse each step.
```
- **Confidentiality only (Q323)**: skip the signing step — just symmetric-encrypt the message and wrap the session key in recipient's public key.
- **Authentication + Digital Signature only (Q322)**: skip encryption — just hash, sign with private key, and send the signature along with the (optionally compressed) plaintext.
- **Both together (Q324)**: do signing first, then encrypt the signed message — this is the full PGP flow shown above.

### Group B: SSL — Core & Handshake
SSL (Secure Sockets Layer) sits **between Transport and Application layer**. It has 4 sub-protocols: Handshake, Change Cipher Spec, Alert, Record.

| Q# | Question | Ans | Why |
|---|---|---|---|
| 333 | OSI layer SSL operates at | **C: Transport layer** | (More precisely just above it, but this is the book's expected answer.) |
| 331 | SSL security features | **B: Confidentiality, integrity, and authentication** | All three, together — that's the whole point of SSL/TLS. |
| 332 | Crypto algorithm for key exchange/encryption in SSL | **A: RSA** | Classic SSL commonly uses RSA for key exchange. |
| 334 | What handshake phase accomplishes | **B: Exchanging cryptographic keys** | The handshake negotiates algorithms and exchanges the keys used for the rest of the session. |
| 335 | Org responsible for SSL/TLS standardization | **C: IETF** | SSL originated at Netscape but was standardized as TLS by the IETF. |
| 336 | Crypto techniques used by SSL Record Protocol | **D: All** | Book's answer covers asymmetric, symmetric, and hashing together as part of the overall SSL suite. *(Technically the Record Protocol itself mainly applies symmetric encryption + MAC/hashing — asymmetric happens during the Handshake — but answer per book is D.)* |
| 337 | NOT part of SSL handshake protocol | **D: DataTransfer** | ClientHello, ServerHello, Certificate Request are real handshake messages; "DataTransfer" isn't a handshake message — it happens in the Record protocol afterward. |
| 338 | Message confirming handshake is complete | **B: Finished** | Sent by both sides at the end of the handshake. |
| 352 | Purpose of Change Cipher Spec Protocol | **C: To signal the transition to encrypted communication** | A tiny message meaning "from now on, everything is encrypted with the negotiated keys." |
| 353 | Handshake phase involving exchange of crypto params/keys | **B: Key Exchange** | Distinguish this from "Authentication" (verifying identity) and "Cipher Specification" (agreeing on algorithms). |
| 354 | What Record Protocol uses against tampering | **C: MAC (Message Authentication Code)** | MAC ensures data integrity of each record. |
| 349 | Certificate type used to authenticate servers in SSL | **D: Digital Certificate** | General term covering the SSL/X.509 server certificate. |

**Four Phases of SSL Handshake (Q328, 2 marks)**:
1. Establish security capabilities (ClientHello/ServerHello — agree on version, cipher suite, session ID).
2. Server authentication & key exchange (server sends its certificate + key exchange info).
3. Client authentication & key exchange (client responds, may send its own certificate if required).
4. Finish (Change Cipher Spec + Finished messages — handshake complete, encrypted session begins).

### Group C: SSL Alert Protocol
| Q# | Question | Ans | Why |
|---|---|---|---|
| 339 | Alert type on fatal error | **C: Fatal alert** | Signals the connection must be terminated immediately. |
| 340 | Alert used to indicate connection is ending (gracefully) | **B: Close notify alert** | Normal, non-error connection closure. |
| 341 | Purpose of a warning alert | **A: To indicate a non-fatal issue that may affect the connection** | Connection can continue. |
| 342 | "bad_record_mac" meaning | **A: A cryptographic integrity check failure on received data** | The MAC didn't match — data was altered or corrupted in transit. |
| 350 | Alert for untrusted certificate chain | **D: Bad certificate** | *(Note: modern TLS technically uses "unknown_ca" for this specific case, but the book's expected answer is "Bad certificate.")* |
| 351 | Primary purpose of the SSL Alert protocol overall | **C: To communicate security-related alerts and errors** | Its whole job — signaling problems (fatal or warning) during the session. |

**Alert Protocol elaboration (Q329, 3 marks)**: Two levels — **Warning** (non-fatal, session may continue, e.g. `close_notify`, `no_certificate`) and **Fatal** (session must terminate immediately, e.g. `handshake_failure`, `bad_certificate`, `bad_record_mac`, `unexpected_message`).

### Group D: IPsec
| Q# | Question | Ans | Why |
|---|---|---|---|
| 343 | Mode encrypting both IP header and payload | **A: Tunnel mode** | Wraps the entire original packet (header + data) inside a new IP packet — used for gateway-to-gateway VPNs. |
| 344 | Purpose of Authentication Header (AH) | **B: To provide data integrity and authentication** | AH does **not** encrypt payload — only authenticates/integrity-checks it. (Contrast with ESP, which also encrypts.) |
| 345 | Mode for end-to-end encryption between hosts | **C: Transport mode** | Only the payload is protected, original IP header stays — used host-to-host. |
| 346 | Primary purpose of IPsec | **B: To secure communications at the network layer** | It works at IP layer, protecting all traffic regardless of application. |
| 347 | ESP mode used for gateway-to-gateway VPN | **A: Tunnel mode** | Same logic as Q343 — tunnel mode is for site-to-site/gateway VPNs. |
| 348 | SPI field size in AH | **B: 32 bits** | Memorize this number. |
| 355 | Key management protocol for IPsec | **A: IKE** | Internet Key Exchange — negotiates and manages the keys IPsec uses. |

**AH vs ESP (for Q358/359 diagrams)**:
```
AH (Authentication Header):
[IP Header][AH][TCP/UDP][Data]     — authenticates header+data, NO encryption

ESP (Encapsulating Security Payload):
[IP Header][ESP Header][TCP/UDP][Data][ESP Trailer][ESP Auth]
                        └──── encrypted ────┘
                 └────────── authenticated ──────────┘
```

**Two modes (Q360, 3 marks)**:
- **Transport mode**: protects only the payload (upper-layer data); original IP header untouched. Used for end-to-end host communication.
- **Tunnel mode**: the *entire* original IP packet is encrypted/authenticated and encapsulated inside a new IP packet with a new header. Used for VPNs between gateways/networks.

**IPsec protocols & applications (Q356/357, 2 marks)**: Protocols = **AH** and **ESP**, managed via **IKE**. Applications: secure branch office connectivity (site-to-site VPN), secure remote access (remote-access VPN), securing extranet/intranet connectivity, enhancing e-commerce security.

---

# PART 3 — UNIT 10: VLAN, VPN

## Topics To Study
- VLAN: definition, OSI layer, types (Default, Data, Voice, Management, Native)
- VPN: definition, how it works, types (Remote Access vs Site-to-Site), OpenVPN

## Prerequisite Concept for This Unit
**VLAN** = logically segments a *single physical* LAN into multiple isolated broadcast domains (no new cabling needed) — a Data Link layer (Layer 2) concept.
**VPN** = creates a secure, encrypted "tunnel" over an *existing* public network (like the internet) so remote traffic looks like it's on a private network — works at the Network layer, often IPsec/SSL-based (ties directly back into Unit 9!).

## MCQ Solutions

### VLAN
| Q# | Question | Ans | Why |
|---|---|---|---|
| 361 | VLAN stands for | **B: Virtual Local Area Network** | Straight definition. |
| 362 | VLAN operates at which OSI layer | **B: Data Link Layer** | Layer 2 — VLANs tag/segment traffic at the switching layer. |
| 363 | VLAN type designed for user-generated data | **B: Data VLAN** | Carries regular user traffic, as opposed to Voice or Management VLANs. |
| 364 | VLAN where all ports belong by default | **C: Default VLAN** | Out-of-the-box, every switch port starts in the Default VLAN (commonly VLAN 1). |
| 365 | Primary purpose of a VLAN | **B: To group one or more LANs for communication based on logical connections** | Logical grouping regardless of physical location/wiring. |

### VPN
| Q# | Question | Ans | Why |
|---|---|---|---|
| 366 | How a VPN works | **B: By routing internet traffic through a specially configured remote server** | Traffic is tunneled through a VPN server, hiding the real path/origin. |
| 367 | Main benefit of VPN for remote access | **C: Securely connecting remote users to the corporate network** | Its core enterprise use case. |
| 368 | Main benefit of OpenVPN | **C: It is highly configurable and secure** | Popular open-source VPN protocol known for flexibility + strong SSL/TLS-based security. |
| 369 | VPN technology for connecting remote branch offices securely | **B: Site-to-Site VPN** | Connects entire networks/offices together, as opposed to Remote Access VPN (single user to network). |
| 370 | What data encryption in a VPN tunnel achieves | **C: Protect data confidentiality** | Direct definition of what encryption does. |

*(No separate long-answer/theory questions appear for Unit 10 in this practice book excerpt — if your syllabus includes VLAN/VPN theory questions from elsewhere, the concept summary above covers the core definitions you'd need.)*

---

# BONUS: Unit 7(?) — ElGamal Digital Signature Numericals (Q262–265)

*(Flagged as Unit 7 in your source PDF — verify with your teacher if these are in scope for tomorrow. Solving them anyway since numericals are high-value marks and the method transfers to any similar question.)*

## The Method (memorize this, not the numbers)

**Setup**: prime `q`, generator `α`, private key `XA`, message hash `m`.
**Public key**: `YA = α^XA mod q`

**To Sign** (need one more input: random `K`, where `gcd(K, q-1) = 1`):
1. `S1 = α^K mod q`
2. Find `K⁻¹ mod (q-1)` (modular inverse)
3. `S2 = K⁻¹ × (m − XA×S1) mod (q-1)`
4. Signature = **(S1, S2)**

**To Verify**:
1. `V1 = α^m mod q`
2. `V2 = (YA^S1 × S1^S2) mod q`
3. **Valid if V1 = V2**

---

### Q262: q=19, α=10, XA=16, m=14 (choosing K=5, gcd(5,18)=1 ✓)
- YA = 10^16 mod 19 = **4**
- S1 = 10^5 mod 19 = **3**
- K⁻¹ mod 18 = **11** (check: 5×11=55=54+1 ≡ 1 mod 18 ✓)
- S2 = 11×(14 − 16×3) mod 18 = 11×(−34) mod 18 = **4**
- **Signature = (3, 4)**
- Verify: V1 = 10^14 mod 19 = **16**; V2 = 4³ × 3⁴ mod 19 = 7×5 mod 19 = **16**
- **V1 = V2 = 16 → Valid ✓**

### Q263: q=17, α=8, XA=9, m=12 (choosing K=3, gcd(3,16)=1 ✓)
- YA = 8^9 mod 17 = **8**
- S1 = 8^3 mod 17 = **2**
- K⁻¹ mod 16 = **11** (check: 3×11=33=32+1 ✓)
- S2 = 11×(12 − 9×2) mod 16 = 11×(−6) mod 16 = **14**
- **Signature = (2, 14)**
- Verify: V1 = 8^12 mod 17 = **16**; V2 = 8² × 2¹⁴ mod 17 = 13×13 mod 17 = **16**
- **V1 = V2 = 16 → Valid ✓**

### Q264: q=13, α=2, XA=3, m=11, K=5 (given directly — gcd(5,12)=1 ✓)
- YA = 2^3 mod 13 = **8**
- S1 = 2^5 mod 13 = **6**
- K⁻¹ mod 12 = **5** (check: 5×5=25=24+1 ✓)
- S2 = 5×(11 − 3×6) mod 12 = 5×(−7) mod 12 = **1**
- **Signature = (6, 1)**
- Verify: V1 = 2^11 mod 13 = **7**; V2 = 8⁶ × 6¹ mod 13 = 12×6 mod 13 = **7**
- **V1 = V2 = 7 → Valid ✓**

### Q265: q=23, α=5, XA=6, H(m)=10, K=3 (given directly — gcd(3,22)=1 ✓)
- YA = 5^6 mod 23 = **8**
- S1 = 5^3 mod 23 = **10**
- K⁻¹ mod 22 = **15** (check: 3×15=45=44+1 ✓)
- S2 = 15×(10 − 6×10) mod 22 = 15×(−50) mod 22 = **20**
- **Signature = (10, 20)**
- Verify: V1 = 5^10 mod 23 = **9**; V2 = 8¹⁰ × 10²⁰ mod 23 = 3×3 mod 23 = **9**
- **V1 = V2 = 9 → Valid ✓**

**Exam tip**: if K isn't given, pick any small number coprime to `q-1` (odd numbers work if q-1 is even, as long as they don't share other factors) — 3 or 5 are usually safe, easy choices. Show the `gcd(K, q-1)=1` check explicitly for method marks.

---

## Final Exam Strategy (given you're starting from zero tonight)

**Highest-yield, lowest-effort wins:**
1. **Kerberos facts** (port 88, MIT, AS/TGS roles) — pure recall, 4+ MCQs guaranteed.
2. **X.509 version history** (v2 = unique IDs, v3 = extensions) — appears 3 times across the MCQs.
3. **The 4 public key distribution methods** — reused across 6+ MCQs and 2 theory questions (296–298, 310). Learn this once, it pays off 4-5 times.
4. **SSL's 4 sub-protocols** (Handshake, Change Cipher Spec, Alert, Record) and what each does — backbone of ~15 MCQs in Unit 9.
5. **AH vs ESP, Tunnel vs Transport** — a clean 2x2 that covers 7 IPsec MCQs.
6. If ElGamal numericals are in scope: the 6-step method above is worth full 5 marks each for pure mechanical execution — no memorization needed, just don't make arithmetic slips.

**For theory answers**: write the **bolded structure headings** first (even if rushed), then fill in 1-liners under each. Partial structure beats a half-finished paragraph for partial credit.

Good luck tomorrow.
