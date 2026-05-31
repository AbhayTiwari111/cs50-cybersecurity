# Week 2 — Securing Data 🔐
**CS50 Introduction to Cybersecurity — David Malan (Harvard)**

---

## 🔑 Password Hashing

### The Problem with Plaintext Passwords
- Storing passwords in plaintext is dangerous — one breach exposes everyone
- Exposed passwords enable **credential stuffing** attacks

### What is Hashing?
- Converts a password into a fixed-length **hash value**
- Hash looks random and doesn't reveal the original password
- Server stores the **hash**, not the password

### How Authentication Works
- **Registration:** password → hashed → store username + hash
- **Login:** password → hashed again → compare to stored hash

### Why Hashing Isn't Perfect
| Attack | Description |
|---|---|
| Dictionary attack | Hash every dictionary word, compare against stolen hashes |
| Brute-force attack | Hash every possible combination and compare |
| Rainbow table | Pre-computed table of password → hash pairs for fast lookups |

> ⚠️ **Red Flag:** If a website emails you your actual password on "Forgot Password" — they store it in plaintext. Stop using that service immediately.

### Recommended Hash Functions
- SHA-256, SHA-512 (SHA-2 family)
- SHA-3 family

---

## 🧂 Salting

### The Problem
- Two users with the same password → same hash → attacker knows passwords match

### What is a Salt?
- A **random value** added to the password before hashing
- Each user gets a unique salt → identical passwords produce different hashes
- Salt is stored alongside the hash (not secret)

### How It Works
```
hash = H(password + salt)
stored_value = salt + hash
```

### Example
| User | Password | Salt | Stored Hash |
|---|---|---|---|
| Carol | cherry | 5050 | xxxx... |
| Charlie | cherry | 4949 | yyyy... |

Same password → completely different hashes ✅

---

## 🔐 Cryptography Basics

### Key Terms
| Term | Meaning |
|---|---|
| Plaintext | Original readable message |
| Ciphertext | Encrypted, unreadable output |
| Encrypt | Plaintext → Ciphertext |
| Decrypt | Ciphertext → Plaintext |
| Key | Secret value that configures the cipher |

### One-Way Hash Functions
- Fixed-length output from any input
- Cannot be reliably reversed
- Used for: password storage, digital signatures, data integrity

---

## 🔒 Symmetric Key Encryption
- Both sender and receiver use the **same key**
- Key must be kept secret by both parties

### Caesar Cipher (Simple Example)
- Rotate each letter by key positions in alphabet
- Key space of only 1–25 = easily brute-forced

### Real-World Algorithms
- **AES** (Advanced Encryption Standard) — widely used today
- **Triple DES** — older but still in use

### Core Problem
> How do two parties agree on a shared secret key if they've never communicated before? → **Key Exchange Problem**

---

## 🗝️ Asymmetric / Public Key Encryption
Every user has two mathematically linked keys:
- 🔓 **Public key** — share with everyone
- 🔐 **Private key** — keep secret, never share

### Encryption Flow
```
Encrypt: plaintext + recipient's PUBLIC KEY  → ciphertext
Decrypt: ciphertext + recipient's PRIVATE KEY → plaintext
```

### RSA Algorithm
- Choose two large prime numbers p and q
- Compute n = p × q
- Security relies on the fact that **factoring large numbers is computationally hard**

> ⚠️ Never invent your own cryptographic algorithms. Use vetted, standardized libraries.

---

## 🤝 Key Exchange — Diffie-Hellman
Establishes a shared secret over an **insecure channel**.

### How It Works
1. Agree publicly on generator `g` and large prime `p`
2. Alice picks private `a`, sends `A = g^a mod p`
3. Bob picks private `b`, sends `B = g^b mod p`
4. Alice computes `s = B^a mod p`
5. Bob computes `s = A^b mod p`
6. Both arrive at the same shared secret `s = g^(ab) mod p` — without ever transmitting it!

> Even if an eavesdropper intercepts A and B, they cannot compute s without knowing a or b.

---

## ✍️ Digital Signatures
Proves a message came from you and **wasn't tampered with**.

### Signing
```
hash = H(document)
signature = encrypt(hash, YOUR PRIVATE KEY)
```

### Verifying
```
expected_hash = H(received document)
decrypted_hash = decrypt(signature, SENDER'S PUBLIC KEY)
valid = (expected_hash == decrypted_hash)
```

| | Physical Signature | Digital Signature |
|---|---|---|
| Can be forged? | Yes | Essentially no |
| Security relies on | Handwriting uniqueness | Secrecy of private key |

---

## 🔑 Passkeys (WebAuthn)
Passwordless authentication using public key cryptography.

### Registration
1. Visit site → device prompts fingerprint/face/PIN
2. Device generates public/private key pair
3. Public key sent to site, **private key stays on device**

### Login
1. Site sends a random challenge
2. Device signs challenge with private key
3. Site verifies signature using stored public key ✅

### Advantages
- No passwords to remember, leak, or reuse
- Immune to credential stuffing and phishing
- Private key never leaves your device

---

## 🌐 Encryption in Transit vs End-to-End

### Encryption in Transit
- Data encrypted between you and the server (HTTPS)
- **Server can still read your data**
- Example: Gmail encrypts your connection, but Google can read emails

### End-to-End Encryption (E2EE)
- Only sender and recipient can read the data
- No intermediary can read it
- Examples: **iMessage, WhatsApp, Signal**

> Use E2EE when privacy is critical. Encryption in transit alone is not enough.

---

## 🗑️ File Deletion & Encryption at Rest

### How "Deletion" Actually Works
- Trash/Recycle Bin does **NOT** delete data
- Emptying bin just removes file's location reference
- Actual bits remain until overwritten — **forensically recoverable**

### Secure Deletion
- Overwrite all bits with 0s, 1s, or random data
- On SSDs, secure deletion is harder due to firmware behavior

### Full-Disk Encryption
- All data encrypted at all times
- Decrypted only when you authenticate
- Examples: **FileVault** (macOS), **BitLocker** (Windows)
- Best practice: Enable from day one

---

## 💀 Ransomware
Malicious use of encryption:
1. Attacker gains access to system
2. Encrypts all victim's data with their own key
3. Demands ransom (usually cryptocurrency) for decryption key
4. No guarantee key will be provided after payment

**Targets:** hospitals, cities, universities, corporations

**Defense:** regular backups, network segmentation, patching, user education

---

## ⚛️ Quantum Computing & Cryptography

| | Classical Bit | Qubit |
|---|---|---|
| States | 0 or 1 | 0 and 1 simultaneously |
| n bits | 1 state at a time | 2ⁿ states simultaneously |

### The Threat
- Quantum computers could break RSA and AES exponentially faster
- Shor's algorithm can efficiently factor large numbers

### Current Status
- Quantum computers capable of breaking modern encryption **do not yet exist at scale**
- NIST is developing **post-quantum cryptography** algorithms

---

## 📋 Quick Reference
| Concept | One-liner |
|---|---|
| Hashing | One-way conversion → fixed-length digest |
| Salt | Random per-user value added before hashing |
| Symmetric encryption | Same key to encrypt and decrypt |
| Asymmetric encryption | Public key encrypts, private key decrypts |
| Diffie-Hellman | Establish shared secret over public channel |
| Digital signature | Private key signs, public key verifies |
| Passkey | Passwordless auth via key pair on your device |
| E2EE | Encrypted sender → recipient only |
| Encryption at rest | Data encrypted while stored on device |
| Ransomware | Attacker encrypts your data, demands payment |

---
*Source: CS50 Introduction to Cybersecurity — Week 2 by David Malan*
