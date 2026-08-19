# Test Log

## Unauthenticated application mapping

| Sequence | Action | Observed result | Evidence |
|---:|---|---|---|
| 1 | Opened Customer Feedback without logging in | Form and CAPTCHA available | 06 |
| 2 | Captured `GET /rest/captcha/` | `200 OK`; response included `7+6+9` and `22` | 06 |
| 3 | Submitted `This is cool` with rating `5` and CAPTCHA answer `22` | `POST /api/Feedbacks/`; `201 Created`; `UserId: null` | 07 |
| 4 | Replayed the feedback request in Repeater | Additional feedback record created | 08/09 sequence |
| 5 | Changed `rating` to `500` | `201 Created`; response persisted `rating: 500` | 08 |
| 6 | Changed comment to include an attribution-like name | `201 Created`; comment changed; `UserId` remained null | 09 |
| 7 | Marked the primary feedback request red and its supporting CAPTCHA request yellow | Finding and evidence relationship is easy to relocate in HTTP history | 10 |
| 8 | Opened Photo Wall and mapped the public application-configuration request | Sent `/rest/admin/application-configuration` to Repeater; removed `If-None-Match`; obtained a full `200 OK` body | 15 pending |
| 9 | Inspected `GET /rest/memories/` without authentication | `200 OK`; response included expected photo metadata plus complete nested `User` objects containing sensitive properties | 12 |
| 10 | Sent `/rest/memories/` to Repeater and established a baseline | No `Authorization` header; response reproduced the nested user-property exposure | 13 pending sanitized recapture |
| 11 | Highlighted `/rest/memories/` red and application configuration light blue | Confirmed finding separated from an endpoint retained for further investigation | 14 |
| 12 | Clicked About Us and observed `GET /api/Feedbacks/` | Response reportedly returned the earlier manipulated `rating: 500`, supporting persistence/user-visible impact for F-002 | 16 pending |
| 13 | Opened Terms of Use hyperlink from About Us | Link pointed to `/ftp/legal.md`, an HTTP-accessible Juice Shop content path to investigate later | 17 pending |
| 14 | Highlighted `/api/Feedbacks/` and `/ftp/legal.md` | Feedback endpoint retained for F-002 correlation; FTP legal document retained as informational follow-up | 17 pending |
| 15 | Created a new Juice Shop user account and mapped registration traffic | Observed `/api/SecurityQuestions/`, `/api/Users/`, `/api/SecurityAnswers/`, and application-configuration traffic | 18 pending |
| 16 | Inspected the `POST /api/Users/` response object | Returned user properties create a hypothesis for mass-assignment testing of fields such as `role` or `deluxeToken` | 18 pending |
| 17 | Logged into the new Juice Shop account | `POST /rest/user/login` returned `200 OK` and an authentication token | 19 |
| 18 | Decoded the login token in Burp Decoder | JWT payload exposed sensitive user-object claims including a password-hash field | 20 |
| 19 | Sent the search request to Repeater | `GET /rest/products/search?q=` returned `200 OK` with product JSON; retained as informational baseline | 21 |
| 20 | Added Apple Juice to the authenticated basket and inspected basket traffic | Observed server-side basket route with numeric object ID `/rest/basket/6` | 26 recommended |
| 21 | Sent basket request to Repeater and changed ID from `6` to `5` | Another user's basket contents were returned, including `UserId: 16` and product items | 26 recommended |
| 22 | Sent `/rest/basket/{id}` request to Intruder | Configured Sniper numeric payloads 1–10 against the basket ID | 22 |
| 23 | Used Grep Extract on Intruder results | Extracted `UserId` values across multiple successful basket ID responses | 23 |
| 24 | Sent `GET /api/Products/1?d=...` to Repeater | Removed `If-None-Match`; received `200 OK` with product id/name/description/price fields | 24 |
| 25 | Used Intruder and Grep Extract against product IDs | Enumerated product names, descriptions, and prices across numeric product IDs; documented as informational product-catalog baseline, not confirmed authz bypass | 25 |

## Burp organization convention

- Prefix Repeater tabs containing confirmed findings with `*`.
- Highlight the primary finding transaction red.
- Highlight supporting or causally related requests yellow.
- Treat colors as analyst annotations, not automatic severity labels.
- Use evidence-supported names; this case is replay/rating tampering, not authenticated-author impersonation.

## Analyst notes

- The CAPTCHA response exposes the solution to the browser.
- The same CAPTCHA values were accepted across multiple feedback creations in the observed sequence.
- The server did not enforce the UI's expected rating range.
- Anonymous feedback may be intended application functionality; it is not labeled an authorization bypass without an explicit requirement.
- The attribution test changed only comment content. It did not prove modification of a trusted author field.
