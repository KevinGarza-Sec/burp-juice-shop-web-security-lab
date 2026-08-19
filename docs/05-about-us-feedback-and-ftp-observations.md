# About Us Mapping: Feedback Listing and FTP Legal Document

## Status

Operator-observed; screenshots pending upload and validation before public evidence inclusion.

## Workflow

1. Returned to Burp's embedded Chromium browser.
2. Clicked **About Us** in OWASP Juice Shop.
3. Observed a generated request for:

```http
GET /api/Feedbacks/ HTTP/1.1
Host: 127.0.0.1:3000
```

4. Inspected the response and observed that the previously manipulated feedback score was returned back through the application-facing API.
5. Followed the About Us hyperlink:

```text
Check out our boring terms of use if you are interested in such lame stuff.
```

6. Observed that it links to the local Juice Shop FTP-style content path, expected to be represented as:

```http
GET /ftp/legal.md HTTP/1.1
```

7. Marked `/api/Feedbacks/` and `/ftp/legal.md` for further investigation.

## Security interpretation

### `/api/Feedbacks/`

The About Us `GET /api/Feedbacks/` response is important because it can show that the earlier `rating: 500` manipulation was not just reflected in the immediate Repeater response. If the manipulated record appears in this later application-facing response, it strengthens F-002 by proving the invalid value persisted and was returned through normal application behavior.

This should be documented as **persistence and user-visible impact** for the rating-validation issue, not as a separate vulnerability unless the feedback listing exposes additional sensitive data.

### `/ftp/legal.md`

The `/ftp/legal.md` path is a content-discovery observation. The word `ftp` in the path does not automatically mean the browser connected to a separate FTP protocol service. In this context, it appears to be an HTTP-accessible route or static-content path inside Juice Shop.

Do not classify it as a vulnerability yet. It is a good item for later investigation because Juice Shop's `/ftp/` area may contain additional files or access-control lessons.

## Evidence markers

```text
16-proof-of-returned-manipulation-to-user.png
17-informational-findings-highlighted.png
```

## Validation checklist for screenshot 16

Before public inclusion, verify that the screenshot shows:

- `GET /api/Feedbacks/`
- `HTTP/1.1 200 OK`
- The manipulated feedback record
- `rating` or `score` value of `500`
- Enough surrounding context to prove this is a later application-facing response, not only the original Repeater manipulation


## Validation checklist for screenshot 17

Before public inclusion, verify that the screenshot shows:

- `/api/Feedbacks/` highlighted for further analysis or finding correlation
- `/ftp/legal.md` highlighted as an informational/follow-up investigation item
- No response body containing secrets, tokens, emails, or unrelated browsing data


## Suggested Burp annotations

- Red or finding color: `/api/Feedbacks/` if it proves the invalid rating is returned by normal application behavior.
- Light blue or informational color: `/ftp/legal.md` for later content-discovery investigation.

Keep a note that Burp colors are analyst annotations, not automatic severity labels.
