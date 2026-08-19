# F-002 — Feedback Rating Lacks Server-Side Range Enforcement

## Status

Confirmed in the authorized local OWASP Juice Shop lab.

## Endpoint

```http
POST /api/Feedbacks/
```

## Expected behavior

The user interface presents a five-star rating control, establishing an expected rating domain of five discrete values. The API should independently reject values outside its permitted range.

## Observed behavior

A request containing:

```json
"rating": 500
```

received:

```http
HTTP/1.1 201 Created
```

The response reflected and persisted:

```json
"rating": 500
```

while the request remained unauthenticated (`UserId: null`).

## Persistence and user-visible return

During later About Us mapping, the application generated:

```http
GET /api/Feedbacks/
```

The operator observed that the response returned the previously manipulated feedback record with the invalid score. Screenshot validation is pending, but if confirmed, this proves the manipulated value was not limited to the immediate Repeater response. It persisted and was later returned through normal application behavior.

## Impact

An attacker can bypass client-side controls and create malformed rating data. In a real application this could corrupt aggregates, distort reporting, break downstream assumptions, appear in user-facing feedback listings, or undermine trust in customer-feedback metrics.

## Reproduction summary

1. Submit ordinary feedback through the UI.
2. Capture `POST /api/Feedbacks/`.
3. Send the request to Repeater.
4. Change `rating` from `5` to `500`.
5. Send the modified request.
6. Observe `201 Created` and `rating: 500` in the response.
7. Continue normal application mapping and open About Us.
8. Locate `GET /api/Feedbacks/`.
9. Confirm whether the later feedback-listing response returns the manipulated `rating: 500` record.

## Suggested remediation

- Validate the field on the server.
- Accept only integers within the defined range, such as `1 <= rating <= 5`.
- Reject invalid values with an appropriate `400` or `422` response.
- Add automated API tests for zero, negative, fractional, non-numeric, null, and excessive values.

## Classification

- CWE-20: Improper Input Validation

No production severity score is assigned because the target is intentionally vulnerable and local.

## Evidence files

- `evidence/screenshots/07-burp-api-feedback-post.png`
- `evidence/screenshots/08-burp-rating-manipulation.png`
- `evidence/screenshots/16-proof-of-returned-manipulation-to-user.png` — pending upload and validation
