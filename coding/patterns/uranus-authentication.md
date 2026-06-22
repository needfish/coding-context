# ------------------------------------------------------------------------------
#  uranus-authentication.md
#  Object variation -- the authentication provider changes while the verification contract stays the same.
# ------------------------------------------------------------------------------

# - Problem -

A user, service, or process claims an identity. Your system needs to verify that claim before letting them do anything.

# - When You'll See This -

Any security boundary -- API endpoints, inter-service calls, file access, privileged UI actions. Authentication answers "who is this?" It's separate from authorisation, which answers "what are they allowed to do?" Authentication gates access to routes, persistence, and commands.

# - How It Works -

- **Actor**: the entity claiming an identity -- a human user, another service, a background process.
- **Credential**: the proof of identity -- a password, API token, certificate, or biometric.
- **Verifier**: the mechanism that checks the credential against a trusted source.
- **Identity store**: the authoritative database of identity information -- a user database, LDAP, OAuth provider.

# - What Must Always Be True -

- The same valid credential always authenticates as the same actor.
- An invalid credential never authenticates as any actor.
- Credentials are stored in a form that can't be reversed (one-way hashing).
- The verification process doesn't leak why it failed (no "username not found" vs "wrong password" distinction).
- A session token is bound to the actor and verifiable without re-sending the credential.

# - What Goes Wrong -

- **Credential leakage**: passwords or tokens are transmitted or stored in plain text.
- **Timing side channel**: the verifier returns faster for "user exists" than "user doesn't exist", allowing attackers to enumerate valid usernames.
- **Token theft**: a valid session token is intercepted and reused by someone else.
- **Session fixation**: an attacker forces a known session token onto a legitimate user.
- **Identity store compromised**: the user database itself is breached.
- **Revocation delay**: a user's access is revoked but their cached token still works.
- **Credential rotation failure**: a user's password expires and the system locks them out because the new password hasn't propagated everywhere.

# - Design Trade-Offs -

- **Convenience vs. security**: passwords are easy but easy to compromise; multi-factor auth is secure but annoying.
- **Stateless vs. stateful tokens**: JWT tokens contain everything needed for verification but can't be revoked early; server-side sessions can be revoked but need a database lookup.
- **Centralised vs. federated**: one auth server is simple to manage but a single point of compromise; OAuth distributes trust but adds protocol complexity.
- **Caching vs. freshness**: caching verification results is faster but risks accepting a recently-revoked credential.

# - Related Problems -

- **saturn/routing**: authentication gates routing decisions.
- **mars/persistence**: data access may require authentication.
- **jupiter/validation**: credentials need validation before verification.
- **mercury/display**: the UI may show different content based on who's logged in.

# - Getting Started -

Hard-code a single API key checked at the entry point. `if request.headers.get("Authorization") != expected_key: return 401`. One key, one check, no database.

# - When to Split Into Its Own Module -

- You have more than one user with different credentials.
- You need to store and verify passwords securely (hashing, salting).
- You need session management (login, logout, expiry).
- You support multiple auth providers (password, OAuth, SSO).

# - Signs You're in This Pattern -

- Your code checks a password or token before allowing an action.
- Multiple endpoints repeat the same auth check logic.
- You need to know who made a request but the identity check is mixed in with business logic.
- You're storing passwords anywhere.

# - Common Mistakes -

- Storing passwords in plain text (use bcrypt, argon2, or PBKDF2).
- Building your own cryptographic protocol instead of using a standard one.
- Adding OAuth before you have a second user.
- Checking auth on every request without caching when performance matters.

# - The Core Challenge -

Authentication is a trust problem. Your system trusts the credential, the verifier trusts the identity store, and the user trusts your system with their credential. A breach at any level breaks the whole chain. The challenge is defining the trust boundary and protecting each link.
