# O-001 — Anonymous Feedback and Attribution-Like Comment Text

## Status

Observed behavior; intentionally not overstated as authenticated-author forgery.

## Evidence

The feedback API accepted an unauthenticated comment containing a claimed name and returned:

```json
{
  "comment": "[arbitrary attribution-like text]",
  "UserId": null
}
```

## Defensible conclusion

An anonymous user can control the feedback comment text, including text that looks like an attribution. The evidence does **not** show that the requester modified a trusted `author`, `email`, or `UserId` property. The server continued to identify the record as anonymous with `UserId: null`.

## Why the distinction matters

User-controlled content and trusted identity metadata are different security boundaries. Calling this forged authenticated authorship would exceed the evidence. A stronger identity-forgery claim would require a request containing an application-owned identity field and proof that the server accepted it as trusted metadata.

## Evidence file

- `evidence/screenshots/09-burp-attribution-text-manipulation.png`
