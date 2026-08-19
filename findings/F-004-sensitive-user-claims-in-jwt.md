# F-004 — Sensitive User Claims Exposed in Client-Readable JWT

## Status

Confirmed in the authorized local OWASP Juice Shop lab.

## Affected workflow

```http
POST /rest/user/login HTTP/1.1
Host: 127.0.0.1:3000
```

## Observed behavior

After logging in, the server returned:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

The response included an authentication token. Returning an authentication token after login is normal for token-based authentication.

The issue is that the token was a client-readable JWT whose decoded payload contained an embedded user object with sensitive account properties. The decoded payload included field names such as:

- `id`
- `email`
- `password`
- `role`
- `deluxeToken`
- `lastLoginIp`
- `profileImage`
- `totpSecret`
- `isActive`
- account timestamps

The visible `password` value was a password hash. A password hash should not be returned to the browser inside a login response or embedded in a JWT payload.

## Important JWT interpretation

JWT payloads are usually Base64URL-encoded and signed, not encrypted. Anyone holding the token can decode the header and payload without the signing key.

Therefore, sensitive values must not be placed in JWT claims unless the token is additionally encrypted and the design truly requires those values. This application does not need to expose password hashes or security metadata to the client to maintain a login session.

## Security classification

- **Primary:** Sensitive information exposure in client-readable token claims
- **CWE:** CWE-200 — Exposure of Sensitive Information to an Unauthorized Actor
- **Related:** CWE-522 — Insufficiently Protected Credentials
- **Lab assessment:** Confirmed

## Impact

In a real application, this pattern could:

- Expose password hashes to the client unnecessarily
- Enable offline password-cracking attempts if the token is stolen
- Reveal roles and account metadata useful for targeting
- Leak security-state fields such as token or MFA-related properties
- Increase the impact of XSS, browser-extension compromise, local malware, or token logging

This lab does not claim that a password was cracked, that the JWT signature was bypassed, or that another account was compromised.

## Reproduction summary

1. Register or use a lab account.
2. Log in through the Juice Shop browser session.
3. Locate:

   ```http
   POST /rest/user/login
   ```

4. Confirm the response returns an authentication token.
5. Copy the JWT payload/token into Burp Decoder.
6. Decode as Base64URL/JWT text.
7. Observe that sensitive user-object fields are present, including the password-hash field.

## Expected behavior

A client-side JWT should contain only minimum session claims, for example:

```json
{
  "sub": "user-id-or-opaque-subject",
  "iat": 1234567890,
  "exp": 1234569999,
  "scope": "customer"
}
```

It should not include:

```text
password
password hash
deluxeToken
totpSecret
lastLoginIp
profileImage identifiers
complete account objects
```

## Remediation

1. Remove password hashes and sensitive account properties from JWT claims.
2. Use minimal claims such as subject, expiry, issuer, audience, and required authorization scopes.
3. Keep sensitive account properties server-side.
4. Rotate any tokens issued with sensitive claims after remediation.
5. Review logging and telemetry to ensure tokens containing sensitive claims were not stored in logs.
6. Add automated tests that assert forbidden fields are absent from authentication tokens.

## Evidence

- `evidence/screenshots/19-user-login-token-shown.png` — sanitized login response showing token field and `200 OK`
- `evidence/screenshots/20-decoded-base64-password-hash-finding.png` — sanitized Decoder view showing JWT decoding workflow

## Claim boundary

> The login flow returned a JWT that decoded to a payload containing sensitive user-object fields, including a password-hash field. The lab did not attempt to crack the hash, forge the JWT, or compromise another account.
