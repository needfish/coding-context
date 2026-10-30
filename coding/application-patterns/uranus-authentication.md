# uranus -- authentication

## Problem

A system must check who is calling before allowing access. The caller may be a user, another service, or a device. The check method may be a password, a token, a certificate, a biometric scan, or an OAuth provider.

## Input

- A credential (password, token, certificate, API key)
- A store or service that can verify credentials

## Output

- An identity (user ID, role list, permissions) if the credential is valid
- A failure result if the credential is invalid

## Observable Behavior

- A caller presenting a valid credential receives an identity
- A caller presenting an invalid credential receives a failure
- The same credential always produces the same result
- The credential is not written to logs, URLs, or error messages

## Minimal Code

```python
import hashlib, os

def hash_password(password: str) -> str:
    salt = os.urandom(16).hex()
    return salt + ":" + hashlib.sha256((salt + password).encode()).hexdigest()

def verify_password(password: str, stored: str) -> bool:
    salt, hash_val = stored.split(":")
    return hash_val == hashlib.sha256((salt + password).encode()).hexdigest()
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Weak password hashing | Stored passwords can be cracked quickly | Use bcrypt, scrypt, or Argon2 (not MD5, SHA-1) |
| Session fixation | Attacker can force a known session ID on a victim | Regenerate session ID after successful login |
| Timing attack | Authentication takes observably different time for valid vs. invalid credentials | Use constant-time comparison for secrets |
| Token leakage | API key appears in logs, URLs, or client-side code | Never log tokens; use short expiry; restrict token scope |
| No rate limiting | Attacker can try millions of passwords | Rate-limit authentication attempts per IP and per user |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Stateless vs. stateful | Stateless (JWT, signed tokens) needs no server-side session store but cannot revoke individual sessions. Stateful (sessions in Redis/DB) supports revocation but needs storage. |
| Password-based vs. passwordless | Passwords are simple but subject to reuse and phishing. Passwordless (WebAuthn, magic links, OAuth) is more secure but needs more infrastructure. |
| Single vs. multi-factor | Single-factor is simple but weak. Multi-factor (password + TOTP + biometric) is more secure but adds user friction. |

## Start With

Use a well-reviewed authentication library (bcrypt for passwords, JWT library for tokens). Do not write cryptographic code from scratch.

## Split Into Its Own Module When

- More than one authentication method is supported.
- Authentication state (sessions, tokens) must be managed.
- Authentication logic is shared across multiple services or endpoints.
- OAuth, SSO, or federated identity is integrated.
- Rate limiting or brute-force protection is needed.

## Signs This Is Present

- A login form or authentication endpoint exists.
- Tokens, session IDs, or API keys are generated and verified.
- Password hashing (bcrypt, scrypt, Argon2) is used.
- Authentication headers (Authorization, Cookie) are parsed.

## Common Mistakes

- Storing passwords in plaintext or with a weak hash.
- Rolling custom cryptography (use well-reviewed libraries).
- Not rate-limiting authentication attempts.
- Leaking authentication state in logs or error messages.
- Implementing authentication but not authorisation (identity checked but no access control).
- Session management bugs (session not invalidated on logout, session fixation).
