# F-005 — Basket IDOR / Broken Object Level Authorization

## Status

Confirmed in the authorized local OWASP Juice Shop lab, with enumeration evidence captured. A direct Repeater screenshot showing the full basket `5` response and item names is recommended as an additional evidence artifact.

## Affected endpoint

```http
GET /rest/basket/{basketId} HTTP/1.1
Host: 127.0.0.1:3000
```

## Observed behavior

After logging in and adding Apple Juice to the basket, the application requested the authenticated basket endpoint with a numeric identifier:

```http
GET /rest/basket/6
```

The numeric object identifier at the end of the route created a test hypothesis:

> Can an authenticated user change the basket ID and read another user's basket?

The request was sent to Repeater. Changing the basket identifier from `6` to `5` returned another basket's contents. The operator observed basket `5` associated with `UserId: 16`, containing multiple items including Eggfruit Juice and Raspberry Juice.

The request was then sent to Intruder to test enumeration across a small controlled range.

## Intruder workflow

1. Right-click the basket request in Repeater.
2. Send to Intruder.
3. Clear auto-added payload positions.
4. Add a single payload marker around the basket ID:

   ```http
   GET /rest/basket/§5§ HTTP/1.1
   ```

5. Use **Sniper** attack type.
6. Configure numeric payloads:

   ```text
   From: 1
   To: 10
   Step: 1
   ```

7. Disable URL encoding of payload characters for this numeric ID test.
8. Start the attack against the local lab target only.
9. Use **Grep - Extract** to pull the returned `UserId` value from each response.

## Evidence observed

The Intruder results showed multiple basket IDs returned `200 OK`, different response lengths, and extracted `UserId` values. Example observed mappings included different payload IDs producing different owner identifiers, demonstrating that the endpoint returned objects outside the current user's own basket.

One payload returned `304 Not Modified`; this is a caching artifact, not an authorization success/failure by itself. Use status, response length, and response body together when validating object access.

## Security classification

- **Primary:** Broken object-level authorization / IDOR
- **OWASP API Security 2023:** API1 — Broken Object Level Authorization
- **CWE:** CWE-639 — Authorization Bypass Through User-Controlled Key
- **Related CWE:** CWE-862 — Missing Authorization
- **Lab assessment:** Confirmed

## Impact

In a real application, this weakness could allow an authenticated user to:

- Enumerate other users' basket IDs
- View another user's basket contents
- Infer user-to-basket relationships
- Collect product preference or purchase-intent data
- Potentially chain into unauthorized basket modification if write endpoints use the same weak authorization pattern

This lab only confirms unauthorized read access/enumeration. It does not claim order placement, payment manipulation, account takeover, or write access to another user's basket unless separately tested and proven.

## Expected behavior

The server should not authorize basket access based only on a client-supplied numeric route ID. It should verify that the authenticated user owns the requested basket or has a legitimate administrative permission.

Unauthorized basket access should return an appropriate error such as:

```http
403 Forbidden
```

or:

```http
404 Not Found
```

## Remediation

1. Enforce object-level authorization on every basket read and write endpoint.
2. Derive the permitted basket from the authenticated session/user context, not from a client-controlled route parameter alone.
3. Return only the current user's basket unless the caller has explicit administrative authority.
4. Add regression tests for cross-user basket IDs.
5. Log and alert on sequential object-ID enumeration attempts.
6. Consider non-sequential identifiers as defense-in-depth, but do not rely on obscurity instead of authorization.

## Verification criteria

After remediation:

- User A can access only User A's basket.
- User A cannot read User B's basket by changing `/rest/basket/{id}`.
- Sequential basket-ID enumeration returns `403`/`404` for unauthorized objects.
- Basket write/update endpoints enforce the same ownership checks.

## Evidence

- `evidence/screenshots/22-intruder-payload.png` — sanitized Intruder numeric payload setup against `/rest/basket/{id}`
- `evidence/screenshots/23-filtered-userid-payload-results.png` — Intruder results with extracted `UserId` values
- Recommended follow-up: `evidence/screenshots/26-basket-idor-direct-repeater-response.png` — direct Repeater response showing basket `5` contents returned to the current authenticated user

## Claim boundary

> Changing the basket ID allowed the authenticated lab user to enumerate and view basket objects associated with other users. The lab did not attempt payment manipulation, order placement, or unauthorized modification of another user's basket.
