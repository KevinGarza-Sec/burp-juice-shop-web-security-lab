# F-001 — Insufficient Anti-Automation: CAPTCHA Answer Exposure and Replay

## Status

Confirmed in the authorized local OWASP Juice Shop lab.

## Affected workflow

```http
GET /rest/captcha/
POST /api/Feedbacks/
```

## Evidence

The unauthenticated CAPTCHA endpoint returned the arithmetic problem and its answer together:

```json
{
  "captchaId": 0,
  "captcha": "7+6+9",
  "answer": "22"
}
```

The captured feedback request used the returned values. Replaying the request more than once produced separate `201 Created` responses and new feedback record IDs while preserving the same CAPTCHA values.

## Impact

An automated client can read the answer directly and reuse the observed values, reducing the control's ability to distinguish human interaction from scripted abuse. In a real application this could increase feedback spam or automated submission volume.

## Reproduction summary

1. Remain unauthenticated.
2. Visit Customer Feedback.
3. Capture `GET /rest/captcha/`.
4. Observe the returned `answer` field.
5. Submit feedback and send the request to Repeater.
6. Replay the same CAPTCHA values.
7. Observe additional `201 Created` responses and distinct record IDs.

## Suggested remediation

- Generate and retain the answer only on the server.
- Bind challenges to a session or signed nonce.
- Expire challenges rapidly.
- Invalidate a challenge after one successful use.
- Rate-limit anonymous submissions.

## Classification notes

Potentially related weakness classes include exposure of sensitive control data and reliance on client-visible validation material. No production severity score is assigned because this is an intentionally vulnerable training application.

## Evidence files

- `evidence/screenshots/06-burp-rest-captcha.png`
- `evidence/screenshots/08-burp-rating-manipulation.png`
- `evidence/screenshots/09-burp-attribution-text-manipulation.png`
- `evidence/screenshots/10-burp-highlighted-findings.png`
