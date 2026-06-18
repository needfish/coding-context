# uranus: Authenticate

## Problem

An actor (user, service, process) must prove its identity before being allowed to perform an action, and the system must verify that proof.

## Context

Arises at every security boundary — API endpoints, inter-service calls, file access, UI actions. Authentication determines who the actor is; it is distinct from authorisation (what the actor is allowed to do). Authentication is rarely a terminal subproblem — it gates access to routes (saturn), persistence (mars), or commands.

## Domains Involved

- **Actor** (biddable or causal): the entity claiming an identity. A human user (biddable), a service (causal), a process (causal).
- **Credential** (lexical): the proof of identity presented by the actor. A password, a token, a certificate, a biometric.
- **Verifier** (causal): the mechanism that checks the credential against an authority.
- **Identity store** (causal): the authoritative source of identity information. A database, an LDAP directory, an OAuth provider.

## Shared Phenomena

- Actor → System: claimed identity, credential
- System → Verifier: credential, verification request
- Verifier → Identity store: lookup request
- Identity store → Verifier: stored credential, status
- Verifier → System: verification result (valid/invalid), actor attributes
- System → Actor: session token (if successful), error (if failed)

## Invariants

- A valid credential must produce a consistent identity — the same credential always authenticates as the same actor.
- An invalid credential must never authenticate as any actor.
- Credentials must not be recoverable from the system's stored representation (one-way hashing).
- The verification process must not leak information about why a verification failed (no distinction between "actor does not exist" and "wrong password").
- A session token must be bound to the actor and verifiable without re-presenting the credential.

## Known Failure Modes

- **Credential leakage**: Credentials are transmitted or stored in plain text, allowing interception and replay.
- **Timing side channel**: The verifier returns faster for "actor exists" than "actor does not exist", allowing enumeration.
- **Token theft**: A valid session token is intercepted and reused by a different actor.
- **Session fixation**: An attacker forces a known session token onto a legitimate actor.
- **Authority compromise**: The identity store itself is compromised, allowing arbitrary identity claims.
- **Revocation delay**: An actor's access is revoked but the system continues to accept their cached tokens or sessions.
- **Credential rotation failure**: An actor's credential expires or is rotated, and the system prevents access because the new credential hasn't propagated.

## Forces

- **Convenience vs. security**: Simple authentication (username + password) is easy to use but easy to compromise; multi-factor authentication is more secure but burdensome.
- **Stateless vs. stateful tokens**: Stateless tokens (JWT) contain all information needed for verification but cannot be revoked before expiry; stateful tokens (sessions) can be revoked but require a server-side lookup.
- **Centralised vs. federated identity**: Centralised identity (single authority) is simple to manage but a single point of compromise; federated identity (OAuth, SAML) distributes trust but adds protocol complexity.
- **Caching vs. freshness**: Caching verification results improves performance but risks accepting a credential that was recently revoked.

## Related Subproblems

- **saturn/route**: Authentication gates routing decisions.
- **mars/persist**: Data access may require authentication.
- **jupiter/validate**: Credentials must be validated before verification.
- **mercury/display**: Display may show different content based on authenticated identity.

## Frame Concern

**Trust delegation.** The system delegates identity verification to an external authority (identity store, OAuth provider) or a stored representation (hashed password). Verification depends on an authority the system does not fully control. The concern is how far the verification result can be trusted and for how long — the gap between "verified at request time" and "authorised at action time."
