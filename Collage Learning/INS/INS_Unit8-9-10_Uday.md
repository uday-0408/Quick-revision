# INS Exam Prep — Unit 8 (Key Exchange) · Unit 9 (Network Security) · Unit 10 (Network Designing)

Built from your question bank (Q266–Q370). 75 MCQs + 30 theory Qs total.
Unit 8 = 30 MCQ + 15 theory. Unit 9 = 35 MCQ + 15 theory. Unit 10 = 10 MCQ + 0 theory in this bank
→ **your 10-mark theory will almost certainly come from Unit 8 or Unit 9.** Prioritize those theory answers first.

---

## 0. PREREQUISITES — know these cold before anything else

You can't answer half of Unit 8/9 without these. 10 min refresher:

| Concept | One-line definition | Why it matters here |
|---|---|---|
| Symmetric key crypto | Same key encrypts & decrypts (AES, DES, 3DES) | Used for bulk data (session keys, ESP payload) |
| Asymmetric (public) key crypto | Key pair: public (share) + private (secret). Encrypt with one, decrypt with other (RSA) | Backbone of PKI, X.509, PGP |
| Hash function | One-way, fixed-length digest (SHA-1/256) | Used to build digital signatures & MACs |
| MAC (Message Authentication Code) | Hash + shared secret key → proves integrity + origin | Used in AH, SSL Record Protocol |
| Digital signature | Hash the message, encrypt the hash with sender's **private** key | Anyone with sender's public key can verify → authentication + non-repudiation |
| Session key | Random one-time symmetric key generated per message/connection | Hybrid encryption pattern: encrypt data with session key, encrypt session key with recipient's public key |
| OSI layers relevant here | L2=Data Link (VLAN), L3=Network (IPsec, IP), L4=Transport (SSL/TLS sits here-ish), L7=Application | Layer-placement MCQs are common gotchas |

**The one pattern that repeats everywhere (PGP, SSL, IPsec-ESP):** encrypt the big message with a fast symmetric session key, then encrypt *that small session key* with the recipient's slow-but-secure public key. Understand this once, and PGP confidentiality, SSL key exchange, and half the "how does X work" questions fall into place.

---

## 1. TOPIC PRIORITY LIST (study in this order)

**Unit 8 — Key Exchange** (weight: very high, 30 MCQ + 15 theory)
1. Public key distribution schemes — 4 types (public announcement, public directory, public-key authority, public-key certificate) — **asked 6+ times as MCQ, 4 times as theory**
2. X.509 certificate — structure, fields, version history (v1/v2/v3) — **asked 8+ times as MCQ, 2x theory**
3. PKI — definition, components (CA/RA/DB), how it works, why used — **asked 6 times MCQ, 4x theory**
4. Kerberos — ports, origin, message flow, cross-realm auth — **asked 6 times MCQ, 5x theory**
5. Certificate revocation, CRL — 2 MCQs but easy marks

**Unit 9 — Network Security** (weight: very high, 35 MCQ + 15 theory)
1. PGP — 5 services (auth, confidentiality, compression, email compat, segmentation), especially confidentiality+authentication combined flow — **10 MCQ + 6 theory, the #1 highest-yield theory topic**
2. SSL/TLS — 4 sub-protocols (Handshake, Change Cipher Spec, Alert, Record), handshake phases, alert types — **14 MCQ + 4 theory**
3. IPsec — AH vs ESP, Transport vs Tunnel mode, SPI, IKE — **8 MCQ + 5 theory**

**Unit 10 — VLAN/VPN** (weight: MCQ only, 10 MCQ)
1. VLAN — definition, OSI layer (Data Link), types (Data/Voice/Management/Default/Native)
2. VPN — how it works, benefits, Site-to-Site vs Remote Access, OpenVPN

**If you only have 2 hours:** Kerberos flow diagram + X.509 fields + PKI components + PGP confidentiality-and-authentication diagram + SSL handshake phases + AH vs ESP vs Tunnel/Transport mode. That combination alone covers most likely theory questions and a majority of the MCQs.

---

## 2. UNIT 8 — MCQ ANSWER KEY (Q266–Q295)

| Q# | Topic | Correct Answer | Why / Gotcha |
|---|---|---|---|
| 266 | PKI | PKI = digital certificates + public-key cryptography + certificate authorities providing enterprise-wide security | Memorize this exact definition — it's the trap answer's opposite in several other Qs |
| 267 | X.509 versions | **v2** | v1 = base fields only. v2 adds Issuer + Subject Unique ID. v3 adds Extensions |
| 268 | X.509 fields | "Serial Modifier" is **not** a real field | Real fields: Version, Serial Number, Signature Algorithm ID, Issuer, Validity, Subject, Public Key Info, (v2) Unique IDs, (v3) Extensions, Signature |
| 269 | Key distribution | "Hashing Certificates" is **not** a real scheme | The 4 real schemes: public announcement, public directory, public-key authority, public-key certificate |
| 270 | Key distribution | **Public-Key Certificates** = most secure | No live third-party contact needed each time; CA signs the binding once |
| 271 | Key distribution | **Public-Key Certificates & Public-Key Authority** use timestamps | Certificate: validity timestamp. Authority: timestamp prevents replay during live exchange |
| 272 | Key distribution | **Public-Key Certificates** — timestamp = expiry date | |
| 273 | X.509 | **RSA** | recommended algorithm in X.509 |
| 274 | PKIX | **7 functions** | registration, initialization, certification, key-pair recovery, key-pair update, revocation request, cross-certification |
| 275 | X.509 versions | Extensions added in **v3** | |
| 276 | Cert revocation | **All of the mentioned** | user no longer certified by CA / CA's own cert compromised / user's private key compromised |
| 277 | CRL | **Certificate Revocation List** | not "Cipher Reusable List" (distractor) |
| 278 | Key distribution | Public directory more secure than **Public announcements** | Directory has an admin controlling entries; announcements have zero verification — anyone can broadcast a fake key |
| 279 | Misc | MongoDB → **x.509** certificate auth | |
| 280 | Kerberos | Client requests a **ticket** from KDC | |
| 281 | X.509 | **X.509** defines certificate structure/fields/values | |
| 282 | Key mgmt | Choose extremely random key, use **full key-space** | |
| 283 | Kerberos | Port **88** | classic gotcha — not 80/25/23 |
| 284 | Kerberos | Developed at **MIT** | |
| 285 | Kerberos | Considers Client + KDC + TGS = **All of them** | |
| 286 | X.509 | **X.509** identifies public key certificate format | not X.500 (that's the directory service standard X.509 sits inside) |
| 287 | Key distribution | USENET → **Public announcements** scheme | |
| 288 | PKI | Elements = CA + RA + Certificate DB = **All of them** | |
| 289 | Auth tokens | **Smart card** | |
| 290 | Virus | Floppy/CD/email = **All of these** | |
| 291 | Confidentiality | Data must be **Encrypted** | |
| 292 | X.509 | Issuer unique ID added in **v2** | same version as subject unique ID (Q267) |
| 293 | PKI | Issuer of PKI = **Certificate authorities** | |
| 294 | Public key crypto | **All of the mentioned** (slow, resource-heavy, high computation) | this is *why* hybrid encryption (public key just for the session key) is used everywhere |
| 295 | Kerberos | Must consider: **Kerberos requires a centrally managed database of all user/resource passwords** | KDC = single point of failure — classic "limitation of Kerberos" gotcha |

---

## 3. UNIT 8 — THEORY ANSWERS (Q296–Q310)

### Q296 (3m) / Q298 (3m) — Key distribution: Public-Key Authority & Public-Key Certificate methods

**Public-Key Authority**
A trusted central authority holds a dynamic, live directory of `{user, public key}`. Every time A wants B's key, A must contact the authority directly (not just once).

```
A --1. request B's key + timestamp--> Authority
A <--2. Authority's signed reply {B's key, timestamp}-- Authority
A --3. message + nonce1 (encrypted w/ B's key)--> B
B --4. request A's key + timestamp--> Authority
B <--5. Authority's signed reply {A's key, timestamp}-- Authority
B --6. nonce1, nonce2 (encrypted w/ A's key)--> A
A --7. nonce2 (encrypted w/ B's key)--> B
```
Tighter security (timestamps stop replay), but the authority is a bottleneck and single point of failure — must be online for every new exchange.

**Public-Key Certificate**
Each user gets a **certificate** = `{ID, public key}` digitally signed by a trusted CA. Users exchange certificates directly — **no live contact with any third party needed per exchange.**

```
A ----[A's Certificate]----> B      (B verifies A's cert using CA's public key)
A <---[B's Certificate]----- B      (A verifies B's cert using CA's public key)
```
Two required properties: (1) anyone can verify who created it and that the key is genuine/current, (2) only the CA can create/update certificates.

### Q297 (5m) — Four categories of public key distribution (full)

| Scheme | How it works | Weakness |
|---|---|---|
| Public announcement | User broadcasts their public key openly (mailing list, website, keyserver) | Anyone can forge/impersonate — zero verification |
| Publicly available directory | A maintained directory `{name, public key}` with registration & periodic publishing | Directory itself can be tampered with if not secured — single point of attack |
| Public-key authority | Live central authority, contacted per-exchange with timestamps | More secure but must be online constantly — bottleneck |
| Public-key certificate | CA-signed certificate binds identity to key, exchanged directly between users | Best of both — no live third party needed, but relies on trusting the CA |

### Q299 (5m) — Components of PKI

- **Certificate Authority (CA):** issues, signs, and revokes digital certificates; the root of trust
- **Registration Authority (RA):** verifies the identity of the certificate requester *before* the CA issues anything (RA has no signing power itself — it's the vetting middleman)
- **Certificate Database/Repository:** stores issued and revoked certificates (CRL)
- **Certificate Store:** local storage on the user's system for their own cert + private key
- **Key/Certificate Archival:** optional backup of keys for recovery

```
User → RA (verifies identity) → CA (signs & issues certificate) → published to Repository/Directory
```

### Q300 (5m) — Discuss X.509 Certificates

X.509 is the ITU-T standard defining the structure of a digital certificate binding a public key to an identity, signed by a CA.

```
Version
Serial Number
Signature Algorithm Identifier
Issuer Name
Period of Validity (Not Before / Not After)
Subject Name
Subject's Public Key Info (algorithm + key)
Issuer Unique Identifier      ← added in v2
Subject Unique Identifier     ← added in v2
Extensions                    ← added in v3
-----------------------------------------
Signature (CA signs the whole block above)
```
- **v1 (1988):** basic fields only
- **v2:** added Issuer & Subject Unique Identifiers (to allow reuse of names over time)
- **v3 (current, most used):** added Extensions field (key usage, alt names, policy constraints etc.) — this is why v3 is what modern TLS uses

### Q301 (3m) — X.509 format + reasons to revoke before expiry

(Draw the diagram from Q300, then:)

Reasons to revoke a certificate before it expires:
1. The user's **private key** is assumed to be compromised
2. The user is **no longer certified** by this CA (e.g., left the organization)
3. The **CA's own certificate** is assumed to be compromised

### Q302 (3m) — Why is PKI used?

Pure public-key crypto has one unsolved problem: *how do you know a public key genuinely belongs to the person who claims it?* PKI solves this by having a trusted CA vouch for the binding.
- Provides authentication, confidentiality, integrity, and **non-repudiation** at internet scale
- Enables secure web browsing (TLS/HTTPS), signed email, code signing, VPNs, e-commerce
- Removes the need to pre-share keys with everyone you talk to

### Q303 (3m) — How PKI works + CA vs RA

1. User generates a key pair
2. RA verifies the user's real-world identity
3. CA signs and issues the certificate binding public key ↔ identity
4. Certificate is published/distributed
5. Relying parties verify the CA's signature (up a **chain of trust** to a root CA)
6. If compromised, the CA revokes it (CRL/OCSP)

**CA (Certificate Authority):** trusted entity that actually signs, issues, and revokes certificates.
**RA (Registration Authority):** does the identity-checking legwork and forwards verified requests to the CA — it cannot sign certificates itself.

### Q304 (5m) / Q305 (5m) — Kerberos in detail + how it authenticates

Kerberos = ticket-based authentication protocol so a user doesn't send their password to every service. Components: **Client**, **AS** (Authentication Server), **TGS** (Ticket Granting Server) — AS+TGS together = **KDC**, and the **Application Server**.

```
Client              AS                 TGS               App Server
  |--1. ID request-------->|
  |<--2. TGT (encrypted)---|
  |--3. TGT + service req------------->|
  |<--4. Service Ticket-----------------|
  |--5. Service Ticket---------------------------------->|
  |<--6. Authenticated confirmation------------------------|
```
- **Step 1–2:** Client authenticates once to AS (with password-derived key) → gets a **Ticket Granting Ticket (TGT)**, valid for a session, so the password itself is never sent again
- **Step 3–4:** Client presents TGT to TGS to request access to a specific service → gets a **Service Ticket**
- **Step 5–6:** Client presents Service Ticket to the actual Application Server → mutual authentication happens, session key established

Every ticket carries a **timestamp** to prevent replay attacks. This ticket-relay design (don't resend password, use short-lived tickets) is the entire point of Kerberos.

### Q306 (3m) — Kerberos cross-realm (other administrative domains)

Different organizations = different **realms**, each with its own KDC. For cross-realm auth, the two realms' TGSs share an **inter-realm key**.

```
Client(Realm A) → A's AS → A's TGS
                              |
                    (inter-realm ticket)
                              ↓
                          B's TGS → B's Application Server
```
Client gets a special ticket from its home TGS for the *remote* realm's TGS, then uses that to get a normal service ticket in the remote realm.

### Q307 (3m) — Requirements of Kerberos + server roles

**Requirements:** Secure (no plaintext passwords over network), Reliable (highly available, distributed), Transparent (user shouldn't notice it), Scalable (supports many users/services).

**Servers:**
- **AS (Authentication Server):** verifies initial login, issues TGT
- **TGS (Ticket Granting Server):** issues service tickets against a valid TGT
- **Application/Service Server:** grants actual resource access after verifying the service ticket

### Q308 (3m) — Kerberos ↔ HTTP server diagram

Same flow as Q304/305, with the Application Server = a web/HTTP server.

```
[Your Name] → AS: login request
AS → [Your Name]: TGT
[Your Name] → TGS: TGT + "I want HTTP server access"
TGS → [Your Name]: Service Ticket (for HTTP server)
[Your Name] → HTTP Server: Service Ticket
HTTP Server → [Your Name]: access granted / authenticated response
```

### Q309 (2m) — Public key Cryptography vs PKI

| | Public Key Cryptography | PKI |
|---|---|---|
| What it is | A mathematical scheme (RSA, ECC) for encrypt/decrypt or sign/verify using key pairs | An entire trust framework — CAs, RAs, certificates, policies — that manages *who owns which public key* |
| Scope | Algorithm-level | System/infrastructure-level |
| Solves | Encryption/signature math | The "is this key really theirs?" trust problem |

### Q310 (2m) — Public announcement & public directory methods

- **Public announcement:** user broadcasts their key openly (posts it, emails it, puts it on a keyserver). Fast and simple, but **anyone can forge a key and claim to be someone else** — no verification step.
- **Publicly available directory:** a maintained registry of `{identity, public key}` pairs. Entries are registered (often in person or via authenticated channel) and the directory is periodically published/queried. More trustworthy than a bare announcement because an admin controls entries — but the directory itself is now a single point of attack if compromised.

---

## 4. UNIT 9 — MCQ ANSWER KEY (Q311–Q355)

### PGP

| Q# | Correct Answer | Why / Gotcha |
|---|---|---|
| 311 | **Phil Zimmermann** | created PGP in 1991 |
| 312 | **Email Encryption** | primary original use case |
| 313 | **RSA and AES** | (originally RSA+IDEA; modern OpenPGP typically RSA/DSA + AES) |
| 314 | **To revoke a compromised or lost private key** | revocation certificate ≠ decrypting anything |
| 315 | **Private Key** | kept secret; public key is shared |
| 316 | **To distribute public keys** | Keyserver's whole job |
| 317 | **PGP 5.0** | introduced OpenPGP standard support |
| 318 | **Recipient's Public Key** (exam-simplified) | Technically: the *message* is encrypted with a random **session key**; that session key is what gets encrypted with the recipient's public key. Know both — theory questions will want the accurate hybrid version |
| 319 | **By encrypting messages with symmetric-key (session key) encryption** | the session key itself is then RSA-encrypted with recipient's public key |
| 320 | **RSA** | generates the PGP key pairs |

### SSL/TLS

| Q# | Correct Answer | Why / Gotcha |
|---|---|---|
| 331 | **Confidentiality, integrity, and authentication** | the 3 pillars SSL provides |
| 332 | **RSA** | key exchange + encryption |
| 333 | **Transport layer** (per this bank) | technically SSL/TLS sits *between* transport and application — common exam simplification is "transport layer" |
| 334 | **Exchanging cryptographic keys** | handshake also authenticates server & negotiates algorithms |
| 335 | **IETF** | standardized TLS (successor to Netscape's SSL) |
| 336 | **All** (asymmetric + symmetric + hashing) | |
| 337 | **DataTransfer** | not a real handshake message |
| 338 | **Finished** | confirms both sides completed handshake |
| 339 | **Fatal alert** | forces connection termination |
| 340 | **Close notify alert** | graceful termination signal |
| 341 | **To indicate a non-fatal issue that may affect the connection** | warning ≠ fatal |
| 342 | **A cryptographic integrity check failure on received data** | `bad_record_mac` = MAC didn't match → tampering/corruption |
| 349 | **Digital Certificate** | authenticates SSL servers |
| 350 | **Bad certificate** (per this bank) | flags an untrusted/invalid cert chain |
| 351 | **To communicate security-related alerts and errors** | Alert Protocol's whole job |
| 352 | **To signal the transition to encrypted communication** | Change Cipher Spec Protocol |
| 353 | **Key Exchange** phase | where crypto params/keys are actually exchanged |
| 354 | **MAC (Message Authentication Code)** | ⚠️ table formatting in the source PDF was garbled here — double check the exact letter in your printed copy, but the *correct concept* is unambiguous: SSL Record Protocol uses a MAC (computed over the data) to detect tampering; symmetric encryption alone handles confidentiality, not tamper-detection |

### IPsec

| Q# | Correct Answer | Why / Gotcha |
|---|---|---|
| 343 | **Tunnel mode** | encrypts entire original IP packet incl. header |
| 344 | **To provide data integrity and authentication** | AH **never** encrypts — classic trap (don't confuse with ESP) |
| 345 | **Transport mode** | end-to-end host-to-host, original IP header stays |
| 346 | **To secure communications at the network layer** | IPsec = Layer 3 |
| 347 | **Tunnel mode** | ESP gateway-to-gateway VPN |
| 348 | **32 bits** | SPI field size in AH |
| 355 | **IKE** (Internet Key Exchange) | the key management protocol for IPsec |

---

## 5. UNIT 9 — THEORY ANSWERS (Q321–Q330, Q356–Q360)

### Q321 (2m) — PGP techniques + workflow

**Techniques:** Digital signature (SHA-1 + RSA/DSS), Message encryption (session key via AES/CAST-128/3DES), Compression (ZIP — after signing, before encrypting), Radix-64 conversion (makes binary data email-safe ASCII), Segmentation (splits oversized messages).

```
Message → [Sign] → [Compress] → [Encrypt] → [Radix-64] → Transmit
```

### Q322 (2m) — PGP Authentication & Digital Signature

```
SENDER:
Message --SHA-1--> Hash --encrypt with sender's PRIVATE key--> Signature
Send: [Signature + Message]

RECEIVER:
Decrypt Signature using sender's PUBLIC key --> Hash(H1)
Compute SHA-1 of received Message --> Hash(H2)
If H1 == H2 → message is authentic & sender can't deny sending it (non-repudiation)
```

### Q323 (2m) — PGP Confidentiality only

```
SENDER:
Generate random one-time SESSION KEY (symmetric)
Encrypt Message with Session Key (fast, e.g. AES)
Encrypt Session Key with recipient's PUBLIC key (RSA)
Send: [Encrypted Session Key + Encrypted Message]

RECEIVER:
Decrypt Session Key using own PRIVATE key
Decrypt Message using that Session Key
```

### Q324 (2m) / Q326 (3m) — Confidentiality + Authentication combined (highest-yield PGP diagram — learn this one well)

```
SENDER:
1. Message --SHA-1--> Hash --sign w/ sender's private key--> Signature
2. [Signature + Message] --compress (ZIP)--> Compressed block
3. Generate random Session Key
4. Encrypt Compressed block with Session Key (symmetric)
5. Encrypt Session Key with recipient's PUBLIC key
6. Send: [Encrypted Session Key + Encrypted(Compressed[Signature+Message])]

RECEIVER: reverse all 6 steps
- decrypt session key w/ own private key
- decrypt block w/ session key
- decompress
- verify signature w/ sender's public key
```
This single diagram answers Q322, Q323, Q324, Q326 — memorize it once, adapt for whichever sub-question is asked.

### Q325 (2m) — PGP working operation (overview)

| Service | Mechanism |
|---|---|
| Authentication | Digital signature (SHA-1 + RSA) |
| Confidentiality | Session key (symmetric) + RSA-encrypted session key |
| Compression | ZIP, applied after signing |
| Email compatibility | Radix-64 conversion |
| Segmentation | Splits large messages for transport limits |

### Q327 (2m) — SSL protocols + Record Protocol

**4 SSL sub-protocols:** Handshake Protocol, Change Cipher Spec Protocol, Alert Protocol, Record Protocol.

**Record Protocol pipeline:**
```
Application data
  → Fragment (into blocks)
  → Compress
  → Add MAC (integrity)
  → Encrypt (confidentiality)
  → Prepend SSL header
  → Transmit
```

### Q328 (2m) — Four phases of SSL Handshake

1. **Establish security capabilities** — ClientHello / ServerHello (agree on version, cipher suite, random values)
2. **Server authentication & key exchange** — Certificate, ServerKeyExchange, (optional CertificateRequest), ServerHelloDone
3. **Client authentication & key exchange** — Client Certificate (if requested), ClientKeyExchange, CertificateVerify
4. **Finish** — ChangeCipherSpec + Finished messages from both sides

### Q329 (3m) — SSL Alert protocol + alert types

2-byte message: **[Severity: Warning(1) / Fatal(2)] + [Alert Code]**

| Alert | Meaning |
|---|---|
| close_notify | Graceful session end |
| unexpected_message | Received inappropriate message |
| bad_record_mac | Integrity check failed |
| handshake_failure | Couldn't agree on parameters |
| bad_certificate | Certificate corrupted/invalid |
| certificate_expired | Cert past validity |
| certificate_revoked | Cert was revoked |
| certificate_unknown | Unspecified cert issue |

Fatal alerts → connection terminates immediately. Warning alerts → may continue.

### Q330 (3m) — Types of SSL protocol, explain one

List the same 4 (Handshake, Change Cipher Spec, Alert, Record). Explain **Handshake Protocol** in depth:

```
Client                                    Server
  --ClientHello-------------------------->
  <-------------------------ServerHello---
  <----------------------Certificate------
  <------------------ServerKeyExchange----
  <----------------[CertificateRequest]---
  <----------------------ServerHelloDone--
  --[ClientCertificate]------------------>
  --ClientKeyExchange-------------------->
  --[CertificateVerify]------------------>
  --ChangeCipherSpec---------------------->
  --Finished------------------------------>
  <---------------------ChangeCipherSpec--
  <-------------------------------Finished
```
Result: both sides now share a master secret → derive session keys → encrypted communication begins.

### Q356 (2m) — Use of IPsec + one protocol w/ diagram

**Use:** Secures IP traffic at the network layer — confidentiality, integrity, authentication, anti-replay. Foundation of most VPNs.

Pick **AH** as the example (diagram below in Q358).

### Q357 (2m) — IPsec protocols and applications

**Protocols:** AH (Authentication Header), ESP (Encapsulating Security Payload), IKE (key management/exchange).
**Applications:** Secure branch-office connectivity (site-to-site VPN), secure remote access over the internet, secure e-commerce transactions, secure extranet/partner connectivity.

### Q358 (2m) — Authentication Header (AH) w/ diagram

```
| Next Header | Payload Length | Reserved            |
| Security Parameters Index (SPI)                    |
| Sequence Number                                     |
| Authentication Data (ICV — Integrity Check Value)   |
```
Provides: integrity + authentication + anti-replay (via sequence number). **Does NOT encrypt** — no confidentiality. Classic gotcha vs ESP.

### Q359 (2m) — ESP w/ diagram

```
| SPI                                    |
| Sequence Number                        |
| Payload Data (ENCRYPTED)               |
| Padding | Pad Length | Next Header     |
| Authentication Data (ICV, optional)    |
```
Provides: confidentiality (encryption) + integrity + authentication (optional) + anti-replay. This is the main gotcha pair with AH — **ESP encrypts, AH does not.**

### Q360 (3m) — Two modes of operation, explain w/ diagram

```
Transport mode:  [ IP header ][ AH/ESP ][ TCP/UDP ][ Data ]
                 (original IP header stays; only payload protected)
                 → host-to-host, end-to-end

Tunnel mode:     [ New IP header ][ AH/ESP ][ Original IP header ][ TCP/UDP ][ Data ]
                 (entire original packet is encapsulated + protected)
                 → gateway-to-gateway VPNs, remote access VPNs
```

---

## 6. UNIT 10 — MCQ ANSWER KEY (Q361–Q370)

| Q# | Topic | Correct Answer | Why / Gotcha |
|---|---|---|---|
| 361 | VLAN | **Virtual Local Area Network** | not "Logical"/"Linear"/"Access" |
| 362 | VLAN layer | **Data Link Layer (L2)** | classic OSI-layer gotcha |
| 363 | VLAN types | **Data VLAN** | for user-generated traffic |
| 364 | VLAN types | **Default VLAN** | all ports belong here when switch is fresh/on |
| 365 | VLAN purpose | **To group devices for communication based on logical connections** (not physical location) | this is the core VLAN idea — logical not physical grouping |
| 366 | VPN mechanics | **By routing traffic through a specially configured remote server** (encrypted tunnel) | |
| 367 | VPN benefit | **Securely connecting remote users to the corporate network** | |
| 368 | OpenVPN | **Highly configurable and secure** | |
| 369 | VPN type | **Site-to-Site VPN** | connects remote branch offices |
| 370 | VPN encryption | **Protect data confidentiality** | |

*No theory questions for Unit 10 in this bank — if your 10-mark theory somehow references VLAN/VPN, know: VLAN = logical L2 segmentation of a switch; VPN = encrypted tunnel over an untrusted network (Site-to-Site vs Remote-Access, protocols like OpenVPN/L2TP/IPsec).*

---

## 7. FINAL GOTCHA CHEAT-SHEET (read this right before the exam)

- **AH vs ESP:** AH = integrity/auth only, NO encryption. ESP = encryption + optional integrity/auth. If a question mentions "confidentiality" or "encryption" in IPsec context → answer is ESP, not AH.
- **Transport vs Tunnel mode:** Transport = original IP header kept, host-to-host. Tunnel = whole packet wrapped in a new IP header, gateway-to-gateway/VPN.
- **X.509 version additions:** v1 = base. v2 = + Unique IDs (Issuer & Subject). v3 = + Extensions. (Repeated in Q267, Q275, Q292 — free marks if memorized.)
- **Kerberos port = 88**, developed at **MIT**, KDC = AS + TGS.
- **PGP hybrid pattern:** sign with sender's private key → compress → encrypt with session key → encrypt session key with recipient's public key. This single flow answers 4+ theory questions.
- **CA vs RA:** CA signs/issues. RA only verifies identity — cannot sign anything itself.
- **4 public key distribution schemes**, ranked least→most secure: public announcement < public directory < public-key authority ≈ public-key certificate (certificate wins for not needing live third-party contact).
- **SSL 4 sub-protocols:** Handshake, Change Cipher Spec, Alert, Record — know what each one's *job* is (negotiate/signal transition/report errors/actually protect data).
- **VLAN = Layer 2 (Data Link)**, not Layer 3. Easy trap.

Good luck — the theory answers above are written exam-length (2/3/5-mark appropriate). If a specific diagram doesn't render clearly for you, ask and I'll break any single one down further before your test.
