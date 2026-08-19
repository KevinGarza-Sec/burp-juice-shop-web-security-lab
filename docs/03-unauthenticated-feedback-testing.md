# Unauthenticated Feedback and Input-Manipulation Testing

## Objective

Map what an unauthenticated visitor can do through the Customer Feedback workflow, then determine whether the server independently enforces CAPTCHA freshness, rating limits, and author identity.

## Scope and safety

- Target: `http://127.0.0.1:3000`
- Application: OWASP Juice Shop, intentionally vulnerable
- Authentication state: anonymous / `UserId: null`
- Tooling: Burp Suite Community Edition Proxy and Repeater
- No external systems were tested.

## Baseline workflow

1. Opened the Customer Feedback page while signed out.
2. Entered the comment `This is cool`.
3. Selected a five-star rating through the UI.
4. Solved the arithmetic CAPTCHA `7+6+9` as `22`.
5. Captured the associated API traffic in Burp.

### Observed endpoints

```http
GET /rest/captcha/
POST /api/Feedbacks/
```

The CAPTCHA response returned both the challenge and answer:

```json
{
  "captchaId": 0,
  "captcha": "7+6+9",
  "answer": "22"
}
```

The baseline feedback submission returned `201 Created`, `rating: 5`, and `UserId: null`.

## Repeater tests

### Test A — Replay

The captured feedback request was sent to Burp Repeater and replayed. Additional feedback records were created successfully.

### Test B — Rating-range manipulation

The rating was changed from the UI-supported value `5` to `500`:

```json
{
  "captchaId": 0,
  "captcha": "22",
  "comment": "This is cool (anonymous)",
  "rating": 500
}
```

The server returned:

```http
HTTP/1.1 201 Created
```

and reflected `rating: 500` in the newly created record. This confirms that the API did not enforce the expected rating range for this request.

### Test C — Attribution-like comment text

The comment text was altered to include a claimed name. The API accepted the text but returned:

```json
"UserId": null
```

This supports the narrow claim that arbitrary attribution-like text can be stored in an anonymous comment. It does not prove modification of an application-owned author or authenticated user field.

## Burp workspace organization

Confirmed findings were organized in Burp using two analyst-defined conventions:

1. Repeater tabs containing a finding receive a leading `*` so they are easy to locate during testing.
2. The primary finding request is highlighted red in Proxy HTTP history, while supporting or causally related evidence is highlighted yellow.

For this workflow:

- Red: `POST /api/Feedbacks/` returning `201 Created` — the request used for replay and input manipulation.
- Yellow: `GET /rest/captcha/` returning `200 OK` — the supporting request that exposed the CAPTCHA challenge and answer.

These colors are analyst annotations, not Burp-generated severity ratings. Use evidence-supported tab names such as `* Feedback - CAPTCHA Replay + Rating Tampering`; avoid `impersonation` unless a trusted identity field is actually forged.

## Security interpretation

1. **CAPTCHA answer exposure:** Returning the challenge and solution in the same unauthenticated API response undermines its value as an anti-automation control.
2. **CAPTCHA replay:** Multiple successful feedback creations using the same CAPTCHA values indicate the values were reusable during testing.
3. **Missing server-side rating validation:** The API accepted `500` despite the user interface presenting a five-star control.
4. **Anonymous content:** The API accepts anonymous feedback by design in this build; that fact alone is not labeled an authorization vulnerability.

## Remediation concepts

- Do not return CAPTCHA solutions to the client.
- Bind each challenge to a server-side session or nonce, expire it quickly, and invalidate it after one successful use.
- Apply rate limits and abuse monitoring to anonymous submissions.
- Validate `rating` server-side using an allowlist/range such as integers `1` through `5`.
- Derive authenticated identity from trusted server-side session state, never from user-controlled display text.

## Evidence

- `06-burp-rest-captcha.png`
- `07-burp-api-feedback-post.png`
- `08-burp-rating-manipulation.png`
- `09-burp-attribution-text-manipulation.png`
