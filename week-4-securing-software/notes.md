# CS50 Cybersecurity — Week 4: Securing Software

> **Topic:** Attacks on software (web-based and local) and how to defend against them.

---

## Table of Contents

1. [Phishing via HTML](#1-phishing-via-html)
2. [Cross-Site Scripting (XSS)](#2-cross-site-scripting-xss)
3. [SQL Injection](#3-sql-injection)
4. [Command Injection](#4-command-injection)
5. [Client-Side vs Server-Side Validation](#5-client-side-vs-server-side-validation)
6. [Cross-Site Request Forgery (CSRF)](#6-cross-site-request-forgery-csrf)
7. [Buffer Overflow & Arbitrary Code Execution](#7-buffer-overflow--arbitrary-code-execution)
8. [Defenses & Best Practices](#8-defenses--best-practices)
9. [Vulnerability Tracking Systems](#9-vulnerability-tracking-systems)

---

## 1. Phishing via HTML

### How HTML Links Work

```html
<a href="https://harvard.edu">Harvard</a>
```

- The **visible text** (between the tags) is what the user sees.
- The **`href` value** is where the user actually goes.
- These two values **do not have to match** — and that's the exploit.

### The Attack

```html
<!-- Shows "https://harvard.edu" but sends user to yale.edu -->
<a href="https://yale.edu">https://harvard.edu</a>
```

An adversary can:
1. Show a trustworthy-looking URL as the link text.
2. Set `href` to a lookalike domain (e.g. `harvvard.edu`).
3. Copy the real site's HTML to make the fake site look identical.
4. Steal credentials when the user logs in.

### Defence (User)
- **Hover over links** before clicking — your browser shows the real destination in the bottom-left corner.
- Check the **URL bar** carefully after navigation.
- Be suspicious of raw **IP addresses** in URLs (e.g. `http://192.168.1.1`).

### Defence (Developer)
- Standardise on **one domain** — never send users to different URL formats.
- Never use bare IP addresses for legitimate services.

---

## 2. Cross-Site Scripting (XSS)

**Definition:** An adversary tricks a website into executing JavaScript code that the adversary wrote.

### Why It's Possible

When a server reflects user input back into its own HTML without sanitising it, an attacker can inject `<script>` tags:

```
Search input typed:  <script>alert('attack')</script>
```

If Google blindly inserts this into their page, the browser executes it.

### Reflected XSS

The attack payload travels **in the URL**:

```
https://www.google.com/search?q=%3Cscript%3Ealert('attack')%3C/script%3E
```

- `%3C` = `<` and `%3E` = `>` (URL encoding)
- The server decodes this and injects it into the HTML.
- An adversary can **embed this URL in an email or website** and send it to victims.

More dangerous payload — stealing cookies:

```javascript
alert(document.cookie)
// or worse: send document.cookie to an attacker's server
```

### Stored XSS

The attack payload is **saved to the server's database** (e.g. in an email or comment) and executes for every user who views it.

### Fix: Character Escaping

Replace dangerous characters **before** rendering user input as HTML:

| Character | Escape Sequence |
|-----------|----------------|
| `<`       | `&lt;`         |
| `>`       | `&gt;`         |
| `&`       | `&amp;`        |
| `"`       | `&quot;`       |
| `'`       | `&#x27;`       |

### Fix: Content Security Policy (HTTP Header)

Add this header from the server to block inline scripts:

```
Content-Security-Policy: script-src 'self'
```

- Prevents inline `<script>` tags from executing.
- Only allows JavaScript loaded from **external `.js` files** on the same origin.
- Same principle applies to CSS with `style-src`.

---

## 3. SQL Injection

**Definition:** An adversary injects SQL code into a query by manipulating user input.

### Vulnerable Code (Python + SQL)

```python
query = f"SELECT * FROM users WHERE username='{username}'"
```

### The Attack — Destructive

If the adversary types as their username:

```
malan'; DELETE FROM users; --
```

The resulting query becomes:

```sql
SELECT * FROM users WHERE username='malan';
DELETE FROM users;
--'
```

- `;` ends the first command and starts a new one.
- `DELETE FROM users` wipes the entire users table.
- `--` comments out the remaining syntax.

### The Attack — Authentication Bypass

Password field input:

```
' OR '1'='1
```

The resulting query becomes:

```sql
SELECT * FROM users
WHERE username='malan'
AND password=''
OR '1'='1'
```

- `'1'='1'` is **always true**, so the query returns all users.
- The server logs in the attacker as the first user (often an admin).

### Fix: Prepared Statements

Let the database handle escaping — never build SQL strings manually.

```python
# Use a placeholder (?) instead of string formatting
query = "SELECT * FROM users WHERE username=?"
cursor.execute(query, (username,))
```

- Single quotes in input are automatically escaped to `''`.
- The database **never interprets** user input as SQL code.

> **Rule:** Never build SQL queries by concatenating user input. Always use prepared statements or parameterised queries.

---

## 4. Command Injection

**Definition:** An adversary injects shell commands through a program that calls `system()` or `eval()`.

```python
import os
os.system(f"search {user_input}")
```

If `user_input` = `cats; rm -rf /`, the shell runs both commands.

### Fix

- Use language-specific safe APIs instead of `system()` or `eval()`.
- If you must use them, use the built-in **escaping functions** provided by the library.
- Never pass raw user input to shell commands.

---

## 5. Client-Side vs Server-Side Validation

### The Threat: Developer Tools

Every user can open browser developer tools and **edit the HTML on their machine**:

```html
<!-- Adversary can remove "disabled" to enable the checkbox -->
<input type="checkbox" disabled>

<!-- Adversary can remove "required" to skip the field -->
<input type="text" required>
```

The server **cannot rely on the browser** to enforce these restrictions — the user controls their own browser.

### Key Principle

| Type | Purpose | Trustworthy? |
|------|---------|-------------|
| Client-side validation | Good UX, immediate feedback | ❌ Can be bypassed |
| Server-side validation | Final gatekeeper | ✅ Always enforce this |

> **Rule:** Client-side validation is a convenience. Server-side validation is a requirement. If you can only have one, always choose server-side.

---

## 6. Cross-Site Request Forgery (CSRF)

**Definition:** An adversary tricks a user's browser into making an unintended request to a site where the user is already logged in.

### Attack via GET (Simple URL)

```html
<!-- Embedded in an adversary's page as a hidden image -->
<img src="https://www.amazon.com/buy?product=B07XLQ2FSK">
```

- The browser auto-fetches image `src` values on page load.
- If the user is logged into Amazon, this **buys the product** without them clicking anything.
- **GET requests should never change server state.**

### Attack via POST (Automatic Form Submission)

```html
<form action="https://amazon.com/buy" method="POST">
  <input type="hidden" name="dp" value="B07XLQ2FSK">
  <button type="submit">Buy Now</button>
</form>

<script>
  document.forms[0].submit(); // Submits without user clicking
</script>
```

POST alone does **not** solve the problem — JavaScript can auto-submit forms.

### Fix: CSRF Token

The server generates a **random, secret token** per user session and embeds it in every form:

```html
<form action="https://amazon.com/buy" method="POST">
  <input type="hidden" name="dp" value="B07XLQ2FSK">
  <input type="hidden" name="csrf_token" value="a3f8x92kqp...">
  <button type="submit">Buy Now</button>
</form>
```

- The adversary **cannot know** this token — they'd have to compromise the server.
- If the submitted token doesn't match what the server expects, the request is rejected.
- Tokens can also be sent as **HTTP headers** for JavaScript-heavy apps.

---

## 7. Buffer Overflow & Arbitrary Code Execution

**Definition:** An adversary provides more input than a program allocated space for, overflowing into adjacent memory — including the return address pointer.

### Memory Layout (Simplified)

```
┌─────────────────────┐  ← High addresses
│     Machine Code    │  (the actual program)
│─────────────────────│
│        ...          │
│─────────────────────│
│   Return Address    │  ← "Where to go back after this function"
│─────────────────────│
│   User Input Buffer │  ← Allocated space (e.g. 16 bytes)
└─────────────────────┘  ← Low addresses (stack grows upward)
```

### The Attack

1. Adversary provides input **longer than the buffer**.
2. Overflow reaches the **return address** stored on the stack.
3. Adversary overwrites the return address to point to their **injected attack code**.
4. When the function returns, the CPU jumps to and executes the adversary's code.

### Impact

- Execute any code with the **same privileges** as the running program.
- Delete files, send spam, steal data, bypass software activation (cracking).
- Remote Code Execution (RCE) if the input arrives over a network.

> The name "Stack Overflow" (the website) is a direct reference to this concept.

### Defences

- Use **memory-safe languages** (Python, Java, Rust) that handle bounds checking automatically.
- In C/C++: always check input length before copying into buffers.
- Modern OS features: stack canaries, ASLR (Address Space Layout Randomisation), NX bits.
- Keep software **updated** — many buffer overflow bugs get patched.

---

## 8. Defences & Best Practices

### For Users

| Practice | Why |
|----------|-----|
| Hover over links before clicking | See real destination URL |
| Check the URL bar after navigation | Detect lookalike domains |
| Install software only from official app stores | Digitally signed by trusted publishers |
| Keep software updated | Patches known CVEs |
| Be suspicious of HTTP (not HTTPS) or bare IP addresses | Signs of an untrustworthy site |

### For Developers

| Threat | Defence |
|--------|---------|
| XSS | Escape user input (`<` → `&lt;`, etc.) + Content Security Policy header |
| SQL Injection | Always use prepared statements / parameterised queries |
| Command Injection | Avoid `system()`/`eval()` with user input; use safe APIs |
| Client-side bypass | Always validate **server-side** as well |
| CSRF | Use CSRF tokens in all state-changing forms |
| Buffer Overflow | Bounds-check all input; prefer memory-safe languages |

### Code Signing & App Stores

1. Developer hashes their software and signs the hash with their **private key** → digital signature.
2. App store (Apple/Google/Microsoft) verifies the signature, then signs it with **their** private key.
3. Your device verifies the app store's signature before installation.

This chain ensures the software hasn't been tampered with.

### Package Managers

Tools like `pip`, `npm`, `gem`, `apt` use the same digital signing model for libraries and system packages.

### Bug Bounty Programs

Companies pay researchers to **responsibly disclose** security vulnerabilities before adversaries exploit them. Severity determines payout. This channels offensive skills toward defensive outcomes.

---

## 9. Vulnerability Tracking Systems

| Acronym | Full Name | Purpose |
|---------|-----------|---------|
| **CVE** | Common Vulnerabilities and Exposures | Unique ID for each known vulnerability |
| **CVSS** | Common Vulnerability Scoring System | Severity score (0–10) for prioritisation |
| **EPSS** | Exploit Prediction Scoring System | Probability that a CVE will be exploited in the wild |
| **KEV** | Known Exploited Vulnerabilities (CISA catalog) | Confirmed vulnerabilities being actively exploited |

> **Further Reading:** [OWASP (Open Worldwide Application Security Project)](https://owasp.org) — the definitive resource for web application security.

---

## Quick-Reference: The Golden Rules

1. **Never trust user input.** Sanitise, escape, or validate everything from outside your program.
2. **Never use GET for state-changing operations.** Use POST + CSRF tokens.
3. **Never rely on client-side validation alone.** Always enforce rules server-side.
4. **Use prepared statements.** Never concatenate user input into SQL strings.
5. **Use existing, battle-tested libraries** for security primitives — don't roll your own.
6. **Keep everything updated.** Most exploits target known, patched vulnerabilities.
7. **Defence in depth.** No single measure is sufficient; layer your defences.
