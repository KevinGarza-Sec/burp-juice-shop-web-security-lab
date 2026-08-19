# Screenshot Evidence Index

## Current sequence

| # | Canonical filename | Purpose | Status |
|---:|---|---|---|
| 01 | `01-juice-shop-local-docker-running.png` | Container and loopback-only mapping | Captured on Kali; pending VPS copy |
| 02 | `02-juice-shop-local-application.png` | Juice Shop loaded locally | Captured; pending final curation |
| 03 | `03-burp-proxy-http-history-juice-shop.png` | Proxy history and local target traffic | Captured; pending final curation |
| 04 | `04-burp-review-request-cache-304.png` | `304 Not Modified` review-cache behavior | Captured; pending final curation |
| 05 | `05-burp-anonymous-guest-basket-no-api-write.png` | Guest basket / client-side state observation | Captured; pending final curation |
| 06 | `06-burp-rest-captcha.png` | CAPTCHA endpoint exposes problem and answer | Verified and copied |
| 07 | `07-burp-api-feedback-post.png` | Anonymous feedback accepted with `201` and `UserId: null` | Verified and copied |
| 08 | `08-burp-rating-manipulation.png` | Repeater accepts and persists `rating: 500` | Verified and copied |
| 09 | `09-burp-attribution-text-manipulation.png` | Arbitrary attribution-like comment accepted; `UserId` remains null | Verified and copied |
| 10 | `10-burp-highlighted-findings.png` | Red-highlighted feedback finding linked to yellow-highlighted CAPTCHA evidence | Verified and copied |
| 11 | `11-burp-repeater-starred-finding-tab.png` | Leading-`*` finding-tab convention | Recapture pending |
| 12 | `12-user-object-found.png` | `/rest/memories/` returns nested `User` objects with excessive sensitive fields | Verified and copied |
| 13 | `13-memories-repeater-baseline-request-findings-tabbed.png` | Unauthenticated Repeater baseline for `/rest/memories/` | Recapture pending |
| 14 | `14-memories-application-configuration-findings-highlights.png` | Red Memories finding and light-blue application-configuration investigation marker | Verified and copied |
| 15 | `15-burp-application-configuration-response.png` | Full `200 OK` response from `/rest/admin/application-configuration` after removing conditional cache header | Pending upload and content review |
| 16 | `16-proof-of-returned-manipulation-to-user.png` | `GET /api/Feedbacks/` later returns the manipulated `rating: 500` record | Pending upload and validation |
| 17 | `17-informational-findings-highlighted.png` | `/api/Feedbacks/` and `/ftp/legal.md` highlighted for investigation/finding correlation | Pending upload and validation |
| 18 | `18-user-object-found.png` | Registration `POST /api/Users/` response returns a `User` object, creating a mass-assignment test hypothesis | Pending upload and validation |
| 19 | `19-user-login-token-shown.png` | Login response returns an authentication token to the browser | Verified and copied |
| 20 | `20-decoded-base64-password-hash-finding.png` | Burp Decoder shows JWT payload is client-readable and contains sensitive user claims | Verified and copied |
| 21 | `21-searchbar-repeater-tab.png` | Search endpoint baseline in Repeater for later investigation | Verified and copied |
| 22 | `22-intruder-payload.png` | Burp Intruder Sniper setup against `/rest/basket/{id}` with numeric payloads 1–10 | Verified and copied |
| 23 | `23-filtered-userid-payload-results.png` | Intruder results using Grep Extract to enumerate returned `UserId` values by basket ID | Verified and copied |
| 24 | `24-product-id-repeater-response.png` | Repeater baseline for `GET /api/Products/1?d=...` returning product fields after removing `If-None-Match` | Verified and copied |
| 25 | `25-authz-intruder-results.png` | Intruder/Grep Extract product-ID enumeration results with names, descriptions, and prices | Verified and copied |
| 26 | `26-basket-idor-direct-repeater-response.png` | Direct Repeater proof that changing `/rest/basket/6` to `/rest/basket/5` returns another user's basket contents | Recommended follow-up capture |

## Naming correction

The working name `06-burp-rast-captcha` was normalized to `06-burp-rest-captcha.png` because the captured endpoint is a REST route: `/rest/captcha/`.

The working name `09-burp-author-manipulation` was narrowed to `09-burp-attribution-text-manipulation.png`. The screenshot proves manipulation of the user-controlled `comment` value, not an application-owned author or `UserId` field.

## Publication standard

Only curated screenshots belonging to the authorized local lab should be published. The evidence set is intended to preserve HTTP method, endpoint, status code, and response structure while excluding credential material, session material, personal identifiers, and unrelated browser activity.

Loopback `127.0.0.1` is retained where it clarifies the lab architecture.
