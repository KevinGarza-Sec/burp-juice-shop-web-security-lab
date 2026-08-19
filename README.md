# Burp Suite Web Security Assessment Lab — OWASP Juice Shop

## Executive summary

This portfolio lab documents an authorized web-security assessment of a local OWASP Juice Shop instance using Burp Suite Community Edition. The intentionally vulnerable target ran in Docker and was bound only to `127.0.0.1:3000`, preventing LAN or public exposure.

The lab demonstrates application mapping, HTTP/API analysis, browser-cache troubleshooting, Burp Proxy and Repeater usage, unauthenticated attack-surface testing, request replay, and server-side input-validation analysis.

> **Scope:** Local training target only. This is lab experience, not a production or client penetration test.

## Architecture

```text
Burp embedded Chromium
        |
        v
Burp Proxy / Repeater on Kali
        |
        v
http://127.0.0.1:3000
        |
        v
OWASP Juice Shop Docker container
```

## Verified results

- Deployed the official Juice Shop container with loopback-only exposure.
- Mapped product, review, CAPTCHA, user-state, feedback, quantity, and Socket.IO endpoints.
- Distinguished `304 Not Modified` cache validation from empty or failed API responses.
- Identified that the guest basket can be maintained in browser `sessionStorage`, explaining why an anonymous basket change may not produce a REST write.
- Submitted feedback without authentication and observed `UserId: null` in the server response.
- Observed the CAPTCHA expression and its answer returned together by `GET /rest/captcha/`.
- Replayed the feedback request with the same CAPTCHA values and received additional `201 Created` responses.
- Changed the client-supplied rating from the UI-supported range to `500`; the API accepted and persisted it.
- Tested attribution-like text in the comment field. The server accepted the text, but continued to return `UserId: null`; this does **not** prove that an authenticated author identity was forged.
- Confirmed that unauthenticated `GET /rest/memories/` responses expose complete nested `User` objects, including password-hash, token, role, email, and account-metadata fields that the Photo Wall does not require.
- During About Us mapping, observed `GET /api/Feedbacks/`; screenshot validation is pending to prove it returned the earlier manipulated `rating: 500` through normal application behavior.
- Identified `/ftp/legal.md` from the About Us terms-of-use hyperlink as a content-discovery item for later investigation.
- Mapped the registration workflow and observed that `POST /api/Users/` returns a user object; this is documented as a mass-assignment hypothesis pending over-posting tests against fields such as `role` or `deluxeToken`.
- Logged into the lab account and confirmed the returned JWT is client-readable and contains sensitive user claims, including a password-hash field.
- Captured `GET /rest/products/search?q=` in Repeater as an informational baseline for later input-validation testing.
- Confirmed a basket IDOR/BOLA issue by changing `/rest/basket/6` to another valid basket ID and using Intruder plus Grep Extract to enumerate `UserId` values across basket IDs.
- Used the same Repeater-to-Intruder workflow against `/api/Products/{id}` to enumerate product names, descriptions, and prices; documented this as an informational product-catalog baseline rather than a confirmed authorization bypass.

## Findings and observations

| ID | Title | Result | Evidence |
|---|---|---|---|
| F-001 | Insufficient anti-automation: CAPTCHA answer exposure and replay | Confirmed in local lab | 06, 08, 09, 10 |
| F-002 | Feedback rating lacks server-side range enforcement | Confirmed in local lab | 07, 08 |
| F-003 | Unauthenticated excessive data exposure in Memories API | Confirmed in local lab | 12, 14; 13 recapture pending |
| F-004 | Sensitive user claims exposed in client-readable JWT | Confirmed in local lab | 19, 20 |
| F-005 | Basket IDOR / Broken Object Level Authorization | Confirmed in local lab | 22, 23; 26 recommended |
| O-001 | Anonymous feedback and arbitrary attribution-like comment text | Observed; not claimed as authenticated-author forgery | 07, 09 |
| INFO-005 | Product API enumeration baseline | Informational; not confirmed authorization bypass | 24, 25 |

Detailed records are under [`findings/`](findings/) and [`docs/`](docs/).

### In-progress observation

Unauthenticated access to `GET /rest/admin/application-configuration` returned a full configuration response after removing `If-None-Match` in Repeater. Classification remains pending until the response fields are reviewed for sensitive versus frontend-required data.

## Evidence

The sanitized screenshot catalog is maintained in [`evidence/SCREENSHOT_INDEX.md`](evidence/SCREENSHOT_INDEX.md).

Current canonical evidence:

- `06-burp-rest-captcha.png`
- `07-burp-api-feedback-post.png`
- `08-burp-rating-manipulation.png`
- `09-burp-attribution-text-manipulation.png`
- `10-burp-highlighted-findings.png`
- `12-user-object-found.png` (sanitized)
- `14-memories-application-configuration-findings-highlights.png`
- `16-proof-of-returned-manipulation-to-user.png` (pending validation)
- `17-informational-findings-highlighted.png` (pending validation)
- `18-user-object-found.png` (pending validation)
- `19-user-login-token-shown.png` (sanitized)
- `20-decoded-base64-password-hash-finding.png` (heavily sanitized)
- `21-searchbar-repeater-tab.png` (sanitized)
- `22-intruder-payload.png` (sanitized)
- `23-filtered-userid-payload-results.png`
- `24-product-id-repeater-response.png` (sanitized)
- `25-authz-intruder-results.png`
- `26-basket-idor-direct-repeater-response.png` (recommended follow-up)

## Skills demonstrated

- Burp Proxy HTTP-history analysis
- Burp Repeater request replay and parameter manipulation
- REST endpoint and method mapping
- HTTP status and JSON response interpretation
- Authentication-state analysis
- Client-side versus server-side validation testing
- CAPTCHA/anti-automation control analysis
- Evidence handling, redaction, and defensible reporting

## Claim boundary

This repository intentionally avoids overstating results:

- Anonymous feedback submission alone is not labeled an authorization bypass without a requirement proving authentication was intended.
- Changing text inside `comment` is not described as forging an application-owned author or `UserId` field.
- The accepted `rating: 500` is documented as improper server-side input validation.
- CAPTCHA behavior is documented as answer exposure and successful replay within the local lab.
- The Memories finding is limited to confirmed unauthenticated disclosure; no password hash was cracked, no exposed token was reused, and no account was compromised.
- The JWT finding is limited to sensitive client-readable claims; returning a login token is not itself a vulnerability, and no JWT forgery or account takeover is claimed.
- The basket BOLA finding is limited to unauthorized read access/enumeration unless later testing proves unauthorized modification, checkout, or payment impact.
- The product-ID Intruder results are treated as product-catalog enumeration, not authorization bypass, unless restricted or hidden product data is proven.

## Acknowledgements

Shout-out to **Netsec Explained** on YouTube for the Burp Suite + OWASP Juice Shop walkthrough that helped guide the lab flow: [Master Burp Suite Like A Pro In Just 1 Hour](https://www.youtube.com/live/QiNLNDSLuJY).

This repository is not a transcript or clone of the video; it documents my own authorized local lab execution, evidence handling, findings, and claim boundaries.

## Cleanup

```bash
sudo docker stop juice-shop-lab
```

The container was launched with `--rm`, so stopping it removes the container while leaving the image cached.

## Interview explanation

> I deployed OWASP Juice Shop locally with Docker and restricted the vulnerable service to loopback. I mapped the application through Burp Proxy, correlated UI actions with REST and Socket.IO traffic, and used Repeater to test trust boundaries. I found that the CAPTCHA endpoint exposed both the challenge and answer, that the same values could be replayed, and that the feedback API accepted a rating far outside the UI-supported range. During Photo Wall mapping, I found that an unauthenticated Memories response serialized full nested user records, including password-hash, token, role, and email properties the public UI did not require. After logging in, I decoded the returned JWT and identified sensitive user claims, then tested a numeric basket endpoint and confirmed Broken Object Level Authorization by changing `/rest/basket/{id}` and using Intruder with Grep Extract to enumerate basket owners. I preserved evidence while redacting tokens, hashes, and identifiers, and avoided claiming password cracking, token reuse, checkout abuse, or account compromise.
