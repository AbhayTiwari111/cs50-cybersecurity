Week 3 — Securing Data 🔐  ||
CS50 Introduction to Cybersecurity — David Malan (Harvard)
---

## Table of Contents

- [Wi-Fi Security](#wi-fi-security)
- [HTTP vs HTTPS](#http-vs-https)
- [Threats: MITM & Packet Sniffing](#threats-mitm--packet-sniffing)
- [Cookies & Session Hijacking](#cookies--session-hijacking)
- [TLS & Certificate Authorities](#tls--certificate-authorities)
- [SSL Stripping & HSTS](#ssl-stripping--hsts)
- [VPN](#vpn-virtual-private-network)
- [SSH](#ssh-secure-shell)
- [Ports & Port Scanning](#ports--port-scanning)
- [Penetration Testing](#penetration-testing--ethical-hacking)
- [Firewalls](#firewalls)
- [Proxies & Deep Packet Inspection](#proxies--deep-packet-inspection)
- [Malware](#malware)
- [Botnets & DDoS](#botnets--ddos-attacks)
- [Defenses](#defenses)

---

## Wi-Fi Security

| Type | Description |
|------|-------------|
| **Unsecured** | No encryption; all traffic is visible |
| **Secured (WPA)** | Wi-Fi Protected Access — encrypts traffic between your device and the access point |

- **WPA** (Wi-Fi Protected Access) is the standard for securing Wi-Fi today, and has evolved through multiple versions.
- Encryption is **only** between your device and the nearby wireless access point (not end-to-end through the internet).
- Look for the 🔒 padlock icon on your device.

---

## HTTP vs HTTPS

```
http://www.example.com   ← NOT encrypted
https://www.example.com  ← Encrypted
```

| Protocol | Encrypted | Port |
|----------|-----------|------|
| HTTP     | ❌ No     | 80   |
| HTTPS    | ✅ Yes    | 443  |

- **HTTP** (Hypertext Transfer Protocol) — plain text, no encryption.
- **HTTPS** — HTTP secured with **TLS** encryption.

---

## Threats: MITM & Packet Sniffing

### 🕵️ Machine-in-the-Middle (MITM) Attack

```
Alice ──── [Eve 👀] ──── Bob
          Intercepts &
          can modify traffic
```

- Without encryption, any device between Alice and Bob can **read or modify** traffic.
- Common scenarios: coffee shop Wi-Fi, hotel Wi-Fi, malicious ISP.
- Attacker doesn't need to be on the same network — just within wireless range.

### 📦 Packet Sniffing

- A **packet** = a virtual envelope carrying data across the internet.
- If packets are **unencrypted**, anyone in the middle can open them and see:
  - Search queries
  - Login credentials
  - Credit card numbers
  - Cookies

> ⚠️ Example: An HTTP POST request sending a credit card number is **fully visible** to any sniffer.

---

## Cookies & Session Hijacking

### How Cookies Work

```
1. You log in → Server sends:
   Set-Cookie: session=1234abcd

2. Every request after → Browser sends:
   Cookie: session=1234abcd

3. Server recognizes you without re-login
```

- A cookie acts like a **hand stamp** at an event — proves identity without re-entering credentials.
- Stored temporarily in memory or persistently on disk (days/weeks/years).

### 🚨 Session Hijacking

- If HTTP (not HTTPS) is used, the cookie is sent **in plain text**.
- An attacker who sniffs `session=1234abcd` can **impersonate you** by sending that same cookie to the server.
- **Solution:** Always use HTTPS to encrypt cookies in transit.

---

## TLS & Certificate Authorities

### TLS (Transport Layer Security)

- The protocol that powers **HTTPS**.
- Successor to SSL (Secure Sockets Layer).
- Uses **public key cryptography** (asymmetric encryption) to establish a secure connection.

### How TLS Handshake Works

```
1. Browser visits https://example.com
2. Server sends its digital certificate (contains public key)
3. Browser verifies certificate was signed by a trusted CA
4. Secure encrypted channel established
```

### Digital Certificates (X.509 format)

A certificate contains:
- Website name
- Validity period
- Public key
- **Digital signature** from a Certificate Authority (CA)

### Certificate Authorities (CAs)

- Companies trusted to **sign** website certificates.
- Browser manufacturers (Google, Apple, Microsoft, Mozilla) maintain a **list of trusted CAs** built into their browsers.
- Trust chain: You trust Apple → Apple trusts CA → CA signed example.com → You trust example.com

---

## SSL Stripping & HSTS

### ⚠️ SSL Stripping Attack

An attacker intercepts your initial HTTP request and either:
1. Keeps the connection as HTTP (never upgrades to HTTPS), or
2. Redirects you to a **lookalike HTTPS site** they control

```
You type: example.com
MITM redirects you to: https://exam1e.com  ← l replaced with 1
```

> This is a **homograph attack** — exploiting visually similar characters.

### 🛡️ HSTS (HTTP Strict Transport Security)

Configured by the server to tell browsers: **"Always use HTTPS with me."**

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

| Directive | Meaning |
|-----------|---------|
| `max-age=31536000` | Enforce HTTPS for 1 year (seconds) |
| `includeSubDomains` | Apply to all subdomains too |
| `preload` | Bake domain into browser's source code |

- After the first visit, browser **automatically enforces HTTPS** for all future visits.
- `preload` eliminates even the **first vulnerable request** — your domain is hardcoded into Chrome, Firefox, etc.

---

## VPN (Virtual Private Network)

```
Alice ══════════════════ VPN Server ──── Internet
      (encrypted tunnel)
```

- Encrypts **all** internet traffic between you and the VPN server (more powerful than HTTPS alone).
- Makes your IP address appear to be that of the VPN server.
- Common uses:
  - Corporate remote access
  - Bypassing geographic restrictions
  - Privacy on untrusted networks

> ⚠️ VPN secures data but **does not fully anonymize** you — the VPN provider can still see your traffic and may be subpoenaed.

---

## SSH (Secure Shell)

```bash
$ ssh user@stanford.edu    # Connect securely to remote server
$ date                     # Run commands on the remote machine
```

- Encrypted protocol for **remotely controlling servers**.
- All commands and responses are encrypted end-to-end.
- Standard port: **22**
- Can also be used to create a VPN-like tunnel.

---

## Ports & Port Scanning

### Common Port Numbers

| Port | Service |
|------|---------|
| 22   | SSH |
| 80   | HTTP |
| 443  | HTTPS |

- Ports range from **0 – 65,535**.
- A server "listens" on a port — like a door waiting for visitors.

### 🔍 Port Scanning

- Technique to discover which ports are open on a server.
- Attackers iterate through all ~65,000 ports looking for vulnerable services.
- "Security through obscurity" (running services on non-standard ports) **is not a real defense** — attackers scan all ports.

---

## Penetration Testing / Ethical Hacking

- **Pen Testing:** Authorized attempt to find vulnerabilities in a system **before** real attackers do.
- **Ethical Hacker:** A paid professional whose goal is to break in — legally.

### Red Team vs Blue Team

| Team | Role |
|------|------|
| 🔴 Red Team | Offensive — tries to penetrate/attack |
| 🔵 Blue Team | Defensive — defends & detects attacks |

Common pen testing activities:
- Port scanning
- Brute-forcing passwords
- Social engineering
- Exploiting known vulnerabilities

---

## Firewalls

A firewall is software (or hardware) that filters traffic **in and out** of a network.

### Filtering Methods

| Method | Description |
|--------|-------------|
| **By IP address** | Block/allow specific IPs (e.g., block social media) |
| **By port number** | Block all ports except allowed ones (e.g., only port 443) |
| **Deep Packet Inspection (DPI)** | Open packets and inspect contents (domain names, email content, attachments) |

> ⚠️ DPI effectively acts as a **MITM by design** — corporations use it to monitor employee traffic or scan for malware.

---

## Proxies & Deep Packet Inspection

```
Alice ──── [Proxy Server] ──── Bob
            Inspects, logs,
            allows or blocks
```

- A **proxy** sits between internal and external networks.
- Can log all traffic, block malicious URLs, or scan for malware.
- Companies may **install their own CA** on your device, enabling them to decrypt and inspect even HTTPS traffic.

### 🔗 URL Rewriting

Employers/schools may rewrite all links in emails:

```
Original link:  https://gmail.com
Rewritten link: https://company.com?url=https://gmail.com
```

**Why:** To check if the destination URL is known to be malicious before letting you visit it.  
**Side effect:** The company now logs every link you click.

> ⚠️ If a company issued your device and installed software on it — **all bets are off**. Assume everything may be monitored.

---

## Malware

**Malware** = Malicious Software — software written to cause harm.

### Types

| Type | Description | Requires Human Action? |
|------|-------------|----------------------|
| **Virus** | Attaches to a host file; spreads when user opens infected file or attachment | ✅ Yes |
| **Worm** | Self-propagates across networks; exploits open ports/vulnerabilities | ❌ No |

### What malware can do:
- Delete all files
- Send spam emails
- Mine cryptocurrency
- Exfiltrate your data
- Enlist your machine in a botnet

---

## Botnets & DDoS Attacks

### Botnet

```
Attacker ──── C&C Server
                  │
        ┌─────────┼─────────┐
      Bot1      Bot2      Bot3   (your infected computer!)
```

- Adversary infects thousands of computers with silent malware.
- Issues commands to all infected machines simultaneously.

### DoS vs DDoS

| Attack | Source | Scale |
|--------|--------|-------|
| **DoS** (Denial of Service) | Single attacker | Easily blocked by IP |
| **DDoS** (Distributed DoS) | Thousands of bots | Hard to block — thousands of IPs |

**Goal:** Overwhelm a server with fake requests so legitimate users can't get through.

---

## Defenses

### ✅ Best Practices Summary

| Defense | What It Protects Against |
|---------|--------------------------|
| Use **HTTPS** always | MITM, packet sniffing, session hijacking |
| Enable **HSTS** (server-side) | SSL stripping |
| Use a **VPN** on untrusted networks | MITM, packet sniffing |
| Use **SSH** for remote access | Unencrypted remote control |
| Configure a **firewall** | Port scanning, unauthorized access |
| **Antivirus software** | Known viruses and worms |
| **Automatic updates** | Known vulnerabilities and exploits |
| **Pen testing** | Discover weaknesses proactively |

### ⚠️ Limitations to Know

- **Antivirus** only works against *known* threats — not zero-days.
- **Zero-day attacks** exploit unknown vulnerabilities before patches exist.
- **Security is layered** — no single tool is enough. The goal is to raise the cost for attackers.

> *"Security really is a multipronged approach... it's a layered defense so that you create a gauntlet of defenses that adversaries have to get through."*  
> — David Malan

---

## Key Acronyms Cheatsheet

| Acronym | Full Name |
|---------|-----------|
| HTTP | Hypertext Transfer Protocol |
| HTTPS | HTTP Secure |
| TLS | Transport Layer Security |
| SSL | Secure Sockets Layer (older TLS) |
| HSTS | HTTP Strict Transport Security |
| VPN | Virtual Private Network |
| SSH | Secure Shell |
| CA | Certificate Authority |
| MITM | Machine-in-the-Middle |
| DPI | Deep Packet Inspection |
| DoS | Denial of Service |
| DDoS | Distributed Denial of Service |
| WPA | Wi-Fi Protected Access |
