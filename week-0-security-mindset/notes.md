Week 0 — Securing Accounts 🔐
CS50 Introduction to Cybersecurity — David Malan (Harvard)

🔑 Core Concepts
Authentication vs Authorization

Authentication = Proving who you are (e.g., entering a password)
Authorization = Verifying whether you should have access
These are two different things — first authenticate, then authorize


⚔️ Password Attacks
1. Dictionary Attack

Attacker uses a file containing real words and tries them one by one as passwords
Defense: Never use dictionary words as passwords

2. Brute Force Attack

Software tries every single possible password combination
4-digit PIN = only 10,000 combinations = cracked in milliseconds
Defense: Use long and complex passwords

Password Strength Math
Password TypePossibilitiesTime to Crack4 digits (0-9)10,000Milliseconds4 letters (a-z, A-Z)7 millionFew seconds4 characters (letters+digits+symbols)78 million~1 minute8 characters (letters+digits+symbols)6 QuadrillionPractically impossible

✅ NIST Password Recommendations

Minimum 8 characters required
Allow up to 64 characters (long passphrases = strong passwords)
Block commonly used passwords (e.g., password123)
Block dictionary words, repetitive (aaaa, 1234) and sequential characters
Block context-specific words (e.g., don't allow "gmail" as Gmail password)
No password hints — they leak personal information
No forced periodic password changes — users just do password1 → password2
Rate limiting — lock account after ~10 failed attempts


🛡️ Defenses
Multi-Factor Authentication (MFA / 2FA)
Three types of factors:
FactorTypeExampleKnowledgeSomething you knowPassword, PINPossessionSomething you havePhone, Key FobInherenceSomething you areFingerprint, Face ID

OTP (One-Time Password) — a code that works only once
SMS-based OTP is less secure (vulnerable to SIM Swap attacks)
Better option: Use an authenticator app like Google Authenticator

Password Manager

Generates a unique, strong password for every website
Stores all passwords in encrypted form
Phishing protection — won't auto-fill on fake/phishing sites
Only need to remember one master password — make it very strong!
Examples: Bitwarden (free), 1Password, iCloud Keychain, Google Password Manager

Single Sign-On (SSO)

"Login with Google / Facebook" option
The third-party website never receives your password — only a confirmation
Benefit: Fewer passwords to remember, inherits your existing security

Passkeys (Emerging Technology)

Replaces passwords with a mathematical key pair
Nothing to memorize — device handles authentication automatically
Based on cryptography — covered in detail in Week 2


🎣 Social Engineering & Phishing
Social Engineering

Tricking someone psychologically to reveal sensitive information
Example: "Write down your password" — this is a trick!
Rule: No legitimate service will ever ask for your password

Phishing

Fake emails/websites designed to look real (e.g., a fake Gmail login page)
Always hover over links to check the actual URL before clicking
For sensitive accounts, manually type the URL instead of clicking links
Social media posts asking "What was your childhood favorite song?" = data harvesting for phishing

Key Logger

Malware that records every keystroke you type
Defense: Only log into accounts on your own personal device

Machine-in-the-Middle Attack

A malicious machine intercepts data between you and a website
Defense: Use HTTPS (encryption — covered in Week 2)


💡 Key Takeaways

Security = Raising the cost for the adversary, not making attacks impossible


Use strong, unique passwords — different for every website
Start using a Password Manager today
Enable MFA — prefer app-based OTP over SMS
Stay alert for phishing — always verify URLs
Only use your own personal device to log in
Take baby steps — secure your most important accounts first
