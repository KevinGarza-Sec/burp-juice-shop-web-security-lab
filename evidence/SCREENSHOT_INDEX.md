# Screenshot Evidence Index

## Current sequence

| # | Canonical filename | Purpose | Validation | Redaction |
|---:|---|---|---|---|
| 01 | `01-juice-shop-local-docker-running.png` | Container and loopback-only mapping | Captured on Kali; pending VPS copy | Redact personal terminal identity if shown |
| 02 | `02-juice-shop-local-application.png` | Juice Shop loaded locally | Captured; pending final curation | None expected |
| 03 | `03-burp-proxy-http-history-juice-shop.png` | Proxy history and local target traffic | Captured; pending final curation | Cookies/tokens if visible |
| 04 | `04-burp-review-request-cache-304.png` | `304 Not Modified` review-cache behavior | Captured; pending final curation | None expected |
| 05 | `05-burp-anonymous-guest-basket-no-api-write.png` | Guest basket / client-side state observation | Captured; pending final curation | Socket.IO `sid` if visible |
| 06 | `06-burp-rest-captcha.png` | CAPTCHA endpoint exposes problem and answer | Verified and copied | None required; local CAPTCHA only |
| 07 | `07-burp-api-feedback-post.png` | Anonymous feedback accepted with `201` and `UserId: null` | Verified and copied | No live auth material visible |
| 08 | `08-burp-rating-manipulation.png` | Repeater accepts and persists `rating: 500` | Verified and copied | No live auth material visible |
| 09 | `09-burp-attribution-text-manipulation.png` | Arbitrary attribution-like comment accepted; `UserId` remains null | Verified; personal name redacted in repo copy | Personal name redacted |
| 10 | `10-burp-highlighted-findings.png` | Red-highlighted feedback finding linked to yellow-highlighted CAPTCHA evidence | Verified and copied | None required; loopback-only target and no secrets shown |
| 11 | `11-burp-repeater-starred-finding-tab.png` | Leading-`*` finding-tab convention | Recapture required: current image says `impersonation` and exposes a personal name | Rename tab and remove/redact personal name before capture |
| 12 | `12-user-object-found.png` | `/rest/memories/` returns nested `User` objects with excessive sensitive fields | Verified, sanitized, and copied | Emails, usernames, password hashes, token values, user IDs, and identifier-bearing profile paths redacted |
| 13 | `13-memories-repeater-baseline-request-findings-tabbed.png` | Unauthenticated Repeater baseline for `/rest/memories/` | Recapture required: active tab is named `5`, old `impersonation` label remains, and sensitive values are visible | Rename tabs and redact identity, hash, and token values |
| 14 | `14-memories-application-configuration-findings-highlights.png` | Red Memories finding and light-blue application-configuration investigation marker | Verified and copied | None required; loopback-only endpoints and no response body shown |
| 15 | `15-burp-application-configuration-response.png` | Full `200 OK` response from `/rest/admin/application-configuration` after removing conditional cache header | Pending upload and content review | Review body for secrets, emails, external URLs, and identifiers |
| 16 | `16-proof-of-returned-manipulation-to-user.png` | `GET /api/Feedbacks/` later returns the manipulated `rating: 500` record | Pending upload and validation | Redact personal identifiers, cookies, tokens, or real account data if visible |
| 17 | `17-informational-findings-highlighted.png` | `/api/Feedbacks/` and `/ftp/legal.md` highlighted for investigation/finding correlation | Pending upload and validation | Usually none if only loopback HTTP-history rows are visible |
| 18 | `18-user-object-found.png` | Registration `POST /api/Users/` response returns a `User` object, creating a mass-assignment test hypothesis | Pending upload and validation | Redact email, password fields/values, security answers, cookies, tokens, user IDs, and personal identifiers |
| 19 | `19-user-login-token-shown.png` | Login response returns an authentication token to the browser | Verified, sanitized, and copied | JWT/token, user ID, and email values redacted |
| 20 | `20-decoded-base64-password-hash-finding.png` | Burp Decoder shows JWT payload is client-readable and contains sensitive user claims | Verified, heavily sanitized, and copied | Encoded token, email, user ID, password-hash value, token-like values, IP/profile/timestamp values redacted |
| 21 | `21-searchbar-repeater-tab.png` | Search endpoint baseline in Repeater for later investigation | Verified, sanitized, and copied | Authorization bearer token and token cookie redacted; old tab labels remain visible and should be renamed in future captures |
| 22 | `22-intruder-payload.png` | Burp Intruder Sniper setup against `/rest/basket/{id}` with numeric payloads 1–10 | Verified, sanitized, and copied | Authorization bearer token redacted |
| 23 | `23-filtered-userid-payload-results.png` | Intruder results using Grep Extract to enumerate returned `UserId` values by basket ID | Verified and copied | No tokens, cookies, emails, or hashes visible |
| 24 | `24-product-id-repeater-response.png` | Repeater baseline for `GET /api/Products/1?d=...` returning product fields after removing `If-None-Match` | Verified, sanitized, and copied | Authorization bearer token, token cookie, and session-like values redacted |
| 25 | `25-authz-intruder-results.png` | Intruder/Grep Extract product-ID enumeration results with names, descriptions, and prices | Verified and copied | No tokens, cookies, emails, or hashes visible |
| 26 | `26-basket-idor-direct-repeater-response.png` | Direct Repeater proof that changing `/rest/basket/6` to `/rest/basket/5` returns another user's basket contents | Recommended follow-up capture | Redact Authorization bearer token, token cookie/JWT, user email, account IDs if tied to the lab account |

## Naming correction

The working name `06-burp-rast-captcha` was normalized to `06-burp-rest-captcha.png` because the captured endpoint is a REST route: `/rest/captcha/`.

The working name `09-burp-author-manipulation` was narrowed to `09-burp-attribution-text-manipulation.png`. The screenshot proves manipulation of the user-controlled `comment` value, not an application-owned author or `UserId` field.

## Public-release rule

Before publishing, verify every screenshot for:

- `Authorization` or bearer tokens
- Session cookies and JWTs
- Real emails, usernames, or personal names
- Public IP or VPN/tailnet identifiers
- Unrelated browser history

Loopback `127.0.0.1` is safe to retain. Redact credential-like and identity values—including seeded fictional accounts—while preserving field names needed to prove the finding.
