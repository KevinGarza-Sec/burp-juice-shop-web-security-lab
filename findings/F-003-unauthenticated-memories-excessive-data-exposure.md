# F-003 — Unauthenticated Excessive Data Exposure in Memories API

## Status

Confirmed in the authorized local OWASP Juice Shop lab.

## Affected endpoint

```http
GET /rest/memories/ HTTP/1.1
Host: 127.0.0.1:3000
```

## Authentication state

The Repeater baseline contained no `Authorization` header. The only visible cookies represented interface state such as language and consent banners, not an authenticated session.

## Observed response

The endpoint returned `HTTP/1.1 200 OK` with the expected Photo Wall properties, including:

- Caption
- Image path
- Creation and update timestamps

Each memory record also contained a complete nested `User` object. Visible properties included:

- Internal user identifier
- Username
- Email address
- Password hash
- Role
- `deluxeToken`
- Last-login IP field
- Profile-image path
- TOTP secret field
- Active-state flag
- Account timestamps and deletion state

The Photo Wall does not require these complete account records to render public images and captions.

## Security classification

- **Primary:** Excessive data exposure / sensitive information exposure
- **CWE:** CWE-200 — Exposure of Sensitive Information to an Unauthorized Actor
- **API category:** OWASP API Security Top 10 2023 API3 — Broken Object Property Level Authorization
- **Lab assessment:** Confirmed

This is stronger than merely discovering a public API route. The response exposes account properties that are unnecessary for the public Photo Wall function.

## Impact

In a real application, this pattern could:

- Enable account and role enumeration
- Expose email addresses and other personal data
- Provide password hashes for offline cracking attempts
- Reveal privileged accounts for targeted attacks
- Expose reusable token material or security metadata
- Increase privacy, credential, and account-takeover risk

No password cracking, token reuse, or account compromise was attempted or claimed in this lab. The finding is based on unauthorized disclosure alone.

## Reproduction

1. Open the Photo Wall in an unauthenticated browser session.
2. In Burp Proxy HTTP history, locate:

   ```http
   GET /rest/memories/
   ```

3. Inspect the `200 OK` JSON response.
4. Confirm that each memory includes normal photo metadata and a nested `User` object.
5. Send the request to Repeater.
6. Send an unchanged baseline request without adding authentication.
7. Confirm the full nested user properties are returned again.

## Expected behavior

A public Photo Wall response should use an explicit allowlist or response data-transfer object, for example:

```json
{
  "id": 1,
  "caption": "Public caption",
  "imagePath": "/public/image.jpg",
  "displayName": "Optional public alias"
}
```

It should never serialize password hashes, authentication tokens, TOTP fields, login metadata, or complete internal account records.

## Remediation

1. Replace automatic model serialization with an allowlisted public response schema.
2. Remove the nested `User` object from the public endpoint, or return only an intentionally public display alias.
3. Apply object-property authorization before serializing account-related fields.
4. Ensure password hashes, MFA secrets, tokens, and login metadata cannot be returned by any client-facing API.
5. Add automated tests asserting that forbidden account properties are absent.
6. In a real deployment, rotate any exposed active tokens and evaluate affected credentials.

## Verification criteria

A remediated response should continue returning the photo caption and image path but must omit:

```text
email
password
role
deluxeToken
lastLoginIp
totpSecret
isActive
```

and other internal account metadata not required by the Photo Wall.

## Evidence

- `evidence/screenshots/12-user-object-found.png` — sanitized response showing the nested sensitive field names
- `evidence/screenshots/13-memories-repeater-baseline-request-findings-tabbed.png` — pending sanitized recapture
- `evidence/screenshots/14-memories-application-configuration-findings-highlights.png` — analyst highlighting and investigation triage

## Claim boundary

> The unauthenticated endpoint disclosed password-hash, token, role, email, and account-metadata fields inside nested user objects. This lab did not attempt to crack a hash, reuse a token, or compromise an account.
