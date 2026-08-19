# Registration Mapping and Mass-Assignment Hypothesis

## Status

Operator-observed registration flow; screenshot `18-user-object-found.png` pending upload and validation.

This is currently a **mass-assignment hypothesis**, not a confirmed mass-assignment finding.

## Observed workflow

A new Juice Shop user account was created through the normal registration page while traffic was captured in Burp Proxy HTTP history.

The registration process generated four notable requests:

```http
GET /api/SecurityQuestions/
POST /api/Users/
POST /api/SecurityAnswers/
GET /rest/admin/application-configuration
```

> Note: if Burp shows a typo-like path such as `/rest/admin/spplication-configuration`, verify the exact captured URL before documenting it as final. In prior mapping the endpoint appeared as `/rest/admin/application-configuration`.

## Why the `User` response object matters

The `POST /api/Users/` response reportedly returned a `User` object. That is important reconnaissance because it exposes which properties the server model understands.

If a response shows fields such as:

- `role`
- `deluxeToken`
- `isActive`
- `id`
- `email`
- `password`
- timestamps or account metadata

then those fields become candidates for **over-posting / mass-assignment testing**.

However, seeing fields in a response does not prove mass assignment by itself. Mass assignment is confirmed only if a client can send privileged or server-controlled fields during object creation/update and the server accepts or persists them.

## Defensible classification right now

- **Current status:** registration model reconnaissance
- **Potential issue:** mass assignment / over-posting
- **Not yet confirmed:** privilege escalation or token manipulation

## Safe test plan for the local lab

Use Burp Repeater against the local Juice Shop instance only.

### 1. Preserve a baseline registration request

Send the original registration request to Repeater and name it:

```text
INFO-003 Registration Baseline
```

Capture:

```text
18-user-object-found.png
```

The screenshot should show enough of the `POST /api/Users/` response to identify returned field names, but values must be redacted before publication.

### 2. Test whether `role` is client-controllable

Duplicate the baseline Repeater tab and rename it:

```text
* F-004 Registration Role Overpost Test
```

Create a new unique lab email and add a privileged-looking field to the JSON body, for example:

```json
{
  "email": "lab-user-redacted@example.local",
  "password": "[REDACTED]",
  "passwordRepeat": "[REDACTED]",
  "securityQuestion": {
    "id": 1,
    "question": "Your eldest siblings middle name?"
  },
  "securityAnswer": "[REDACTED]",
  "role": "admin"
}
```

Send the request and inspect whether the response stores `role: "admin"`, ignores it, or rejects the request.

### 3. Test whether `deluxeToken` is client-controllable

Use a new unique lab email and add:

```json
"deluxeToken": "kg-lab-test-token"
```

Confirm whether the response persists, ignores, or rejects the value.

### 4. Verify persistence separately

A same-response reflection is weaker than persistence. If a privileged field appears accepted, verify through a later user-state or user-profile endpoint, or through the UI if visible.

## Evidence hierarchy

1. Original registration request and response shows the model fields.
2. Modified registration request shows the over-posted field sent by the client.
3. Response shows server acceptance or rejection.
4. Follow-up request/UI confirms whether the field persisted.

## Redaction requirements

Before publishing screenshot 18 or any mass-assignment test evidence, redact:

- Registered email address
- Password and repeated password
- Security answer
- Session cookies
- Authorization headers or JWTs
- User IDs if tied to the new lab account
- Tokens such as `deluxeToken`
- Any real personal name or account identifier

Keep field names visible. The point is to prove the server returns or accepts specific properties, not to publish their values.

## Interview explanation

> During registration mapping, I captured the security-question, user-creation, and security-answer requests. The user-creation response returned a user object, which helped identify server-side model properties worth testing for mass assignment. I treated that as a hypothesis until I could over-post fields such as `role` or `deluxeToken` and verify whether the server persisted them.
