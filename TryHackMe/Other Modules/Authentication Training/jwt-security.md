# JWT Security

## Overview

JSON Web Tokens (JWTs) are a widely used standard for token-based session management in APIs and modern web applications. Their power comes from the signature — a cryptographic mechanism that allows any service to verify a token's authenticity without contacting the authentication server. However, when signature verification is implemented incorrectly, JWTs can be forged, granting attackers arbitrary privilege escalation. This room covers the JWT structure, signing algorithms, and six common implementation vulnerabilities with practical exploitation techniques.

---

## Topics Covered

- JWT structure: header, payload, signature
- Signing algorithms: None, HS256 (symmetric), RS256 (asymmetric)
- Sensitive information disclosure in JWT claims
- Not verifying the signature
- Downgrading to the None algorithm
- Weak symmetric secrets and offline cracking
- Algorithm confusion (RS256 → HS256) attack
- Token lifetime and missing `exp` claim
- Cross-Service Relay attack (audience claim)

---

## Key Concepts

### JWT Structure

A JWT is three Base64Url-encoded components separated by dots:

```
header.payload.signature
```

| Component | Contents |
|-----------|---------|
| **Header** | Token type (`JWT`) and signing algorithm (`alg`: `HS256`, `RS256`, `none`) |
| **Payload** | Claims — pieces of data about the user or session (username, role, expiry, etc.) |
| **Signature** | Cryptographic verification of header + payload integrity |

**Decoding:** Base64Url decoding the header and payload reveals their contents in plain JSON — JWTs are not encrypted by default.

---

### Signing Algorithms

| Algorithm | Type | How it works | Security |
|-----------|------|-------------|---------|
| `none` | None | No signature | No integrity verification at all |
| `HS256` | Symmetric | HMAC-SHA256 with a shared secret | Security depends entirely on secret strength |
| `RS256` | Asymmetric | RSA signature with private key; verified with public key | Private key stays secret; public key can be shared |

**Security of the JWT relies entirely on the signature being correctly verified.** The claims in the payload can be modified by anyone — only the signature prevents forgery.

---

## Vulnerabilities and Exploitation

### 1. Sensitive Information Disclosure

JWTs are sent to and stored client-side. Any claims in the payload are visible to anyone who decodes the token. Sensitive data should never be stored in JWT claims.

**Vulnerable implementation:**
```python
payload = {
    "username": username,
    "password": password,       # Exposed to the client
    "admin": 0,
    "flag": "[redacted]"        # Also exposed
}
```

**Fix:** Store sensitive values server-side. Use the verified JWT username as a key to look up sensitive data from the backend database.

---

### 2. Signature Not Verified

If a server decodes and uses JWT claims without verifying the signature, any claim in the payload can be modified to any value.

**Vulnerable implementation:**
```python
payload = jwt.decode(token, options={'verify_signature': False})
```

**Exploitation:** Remove the signature from the JWT (leave only the dot after the payload), modify the `admin` claim to `1`, and submit. The server processes it as valid.

**Fix:**
```python
payload = jwt.decode(token, secret, algorithms=["HS256"])
```

---

### 3. Algorithm Downgrade to None

If the server reads the algorithm from the JWT header and passes it to the decode function without restricting allowed algorithms, an attacker can set `alg: none` — causing most libraries to accept the token without any signature verification.

**Exploitation:**
1. Decode the JWT header and payload
2. Change `"alg": "HS256"` to `"alg": "none"` in the header
3. Re-encode the header and payload (Base64Url)
4. Remove or empty the signature
5. Submit the modified token — server accepts it as valid

**Vulnerable implementation:**
```python
header = jwt.get_unverified_header(token)
signature_algorithm = header['alg']
payload = jwt.decode(token, secret, algorithms=signature_algorithm)
```

**Fix:** Specify only explicitly allowed algorithms as a list:
```python
payload = jwt.decode(token, secret, algorithms=["HS256", "HS384", "HS512"])
```

---

### 4. Weak Symmetric Secret (Offline Cracking)

If HS256 is used with a weak or common secret, the signature can be cracked offline using a wordlist. Once the secret is known, valid tokens with any claims can be forged.

**Cracking with Hashcat:**
```bash
# Save the JWT to jwt.txt
# Download a JWT secret wordlist
wget https://raw.githubusercontent.com/wallarm/jwt-secrets/master/jwt.secrets.list
# Crack with hashcat
hashcat -m 16500 -a 0 jwt.txt jwt.secrets.list
```

Once the secret is recovered, forge a new token:
```python
import jwt
payload = {"username": "user", "admin": 1}
token = jwt.encode(payload, recovered_secret, algorithm="HS256")
```

**Fix:** Use a long, randomly generated secret (at minimum 256 bits). JWT secrets are used by software, not humans — there is no excuse for a weak secret.

---

### 5. Algorithm Confusion (RS256 → HS256)

When both symmetric (HS256) and asymmetric (RS256) algorithms are accepted, some library implementations use the `secret` parameter for HS256 decoding. If the algorithm is downgraded from RS256 to HS256, the library may use the server's **public key** as the HMAC secret.

Since the public key is, by definition, publicly available, the attacker can sign a forged token using HS256 with the public key as the secret.

**Exploitation:**
```python
import jwt
public_key = "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----"
payload = {"username": "user", "admin": 1}
token = jwt.encode(payload, public_key, algorithm="HS256")
```

**Vulnerable implementation:**
```python
payload = jwt.decode(token, secret, algorithms=["HS256", "HS512", "RS256", "RS512"])
```

**Fix:** Separate the algorithm families with explicit logic:
```python
header = jwt.get_unverified_header(token)
algorithm = header['alg']
if "RS" in algorithm:
    payload = jwt.decode(token, public_key, algorithms=["RS256", "RS384", "RS512"])
elif "HS" in algorithm:
    payload = jwt.decode(token, secret, algorithms=["HS256", "HS384", "HS512"])
```

---

### 6. Missing or Excessive Token Lifetime

If the `exp` (expiration) claim is not set or is set to a very large value, the token is effectively permanent. A token issued years ago may still be valid, giving persistent access to anyone who captured it.

**Using a persistent token:**
If a token has no `exp` claim, it can be used indefinitely — even years after issuance.

**Fix:** Always set an `exp` claim appropriate to the application's risk profile:
```python
import datetime
lifetime = datetime.datetime.now() + datetime.timedelta(minutes=5)
payload = {"username": username, "admin": 0, "exp": lifetime}
token = jwt.encode(payload, secret, algorithm="HS256")
```

---

### 7. Cross-Service Relay Attack (Audience Claim)

In SSO environments, the same authentication server issues JWTs for multiple applications. The `aud` (audience) claim indicates which application the token is intended for. If an application does not verify the audience claim, a token issued with elevated privileges for Application B (where the user is an admin) can be used against Application A (where the user should not be an admin).

**Exploitation:**
1. Authenticate to Application B where you have admin privileges — receive a JWT with `"admin": 1` and `"aud": "appB"`
2. Use this token against Application A — if Application A does not verify the audience, it trusts the admin claim

**Fix:** Always verify the audience claim:
```python
payload = jwt.decode(token, secret, audience=["appA"], algorithms="HS256")
```

---

## JWT Vulnerability Summary

| Vulnerability | Attack | Prevention |
|---------------|--------|-----------|
| Sensitive info in claims | Read the decoded payload | Never store sensitive data in JWT claims |
| Signature not verified | Modify any claim freely | Always verify signature with secret/public key |
| None algorithm downgrade | Remove signature, set alg=none | Whitelist allowed algorithms explicitly |
| Weak symmetric secret | Offline hashcat cracking | Use long random secret (256+ bits) |
| Algorithm confusion | Sign HS256 with public key | Separate algorithm family handling |
| Missing exp claim | Reuse old tokens indefinitely | Always set appropriate exp value |
| Missing audience check | Use appB admin token on appA | Verify audience claim on every decode |

---

## Important Terminology

| Term | Meaning |
|------|---------|
| JWT | JSON Web Token — self-contained token encoding claims with a cryptographic signature |
| Claim | A key-value pair in the JWT payload representing information about the entity |
| `exp` | Expiration time claim — UNIX timestamp after which the token is invalid |
| `aud` | Audience claim — identifies which application(s) the token is intended for |
| HS256 | HMAC-SHA256 — symmetric signing algorithm using a shared secret |
| RS256 | RSA-SHA256 — asymmetric algorithm signed with private key, verified with public key |
| Algorithm confusion | Attack exploiting mixed symmetric/asymmetric algorithm support to sign with the public key |
| Cross-Service Relay | Using a token intended for one application against a different application in the same SSO environment |

---

## Real-World Relevance

- JWT algorithm confusion attacks were discovered in real implementations of popular frameworks — the None and RS256→HS256 downgrade attacks have been found in production systems
- Weak JWT secrets (e.g. `secret`, `password`, `changeme`) are commonly found in applications that copied example code from tutorials
- Missing `exp` claims create permanently valid tokens — if a session database is ever compromised, those tokens are valid forever
- Cross-Service Relay attacks are relevant in microservice architectures and SSO deployments where multiple applications share one authentication service
- Hashcat can crack HS256 JWT secrets using GPU acceleration at millions of attempts per second — weak secrets are brute-forced quickly

---

## Key Learnings

- JWT claims are visible to anyone — never store passwords, flags, or sensitive data in claims
- The JWT is only as secure as its signature — always verify it, never skip it
- Explicitly whitelist allowed algorithms — never accept the algorithm claimed in the header without restriction
- Use long, random secrets for HS256 — weak secrets are crackable offline
- Separate HS and RS algorithm handling to prevent algorithm confusion
- Always set an `exp` claim; use a token lifetime appropriate to the application's sensitivity
- In SSO environments, verify the audience claim to prevent cross-service token relay

---

## Conclusion

JWTs derive their security entirely from the signature. Every vulnerability in this room — None algorithm, missing verification, algorithm confusion, weak secrets — bypasses or defeats that signature check. A correctly implemented JWT requires: strong random secrets, explicit algorithm whitelisting, audience verification in SSO contexts, appropriate expiry, and no sensitive data in claims. Getting any of these wrong creates a path to complete session forgery and privilege escalation.
