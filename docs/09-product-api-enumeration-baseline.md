# Product API Enumeration Baseline

## Status

Informational baseline / authorization hypothesis tested. Not classified as a vulnerability based on current evidence.

## Observed workflow

From Proxy HTTP history, the product detail request was sent to Repeater:

```http
GET /api/Products/1?d=Tue%20Aug%2018%202026 HTTP/1.1
Host: 127.0.0.1:3000
```

The request originally contained a conditional cache header:

```http
If-None-Match: W/"..."
```

Removing that header produced a fresh response body:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

The response included product fields such as:

- `id`
- `name`
- `description`
- `price`
- `deluxePrice`
- `image`
- timestamps

## Header interpretation

Removing `If-None-Match` is useful when the browser has a cached ETag and Burp would otherwise receive `304 Not Modified` with no body.

Removing:

```http
Connection: keep-alive
```

is not required for this. It controls connection reuse, not cache validation.

## Intruder workflow

The same numeric-ID workflow used for the basket test was applied to the product endpoint:

1. Send the product request to Intruder.
2. Clear auto-added positions.
3. Add a payload marker around the product ID.
4. Use a numeric payload range.
5. Configure Grep Extract before starting the attack to extract useful fields.

Useful extract fields included:

```text
"name"
"description"
"price"
```

The results showed multiple product IDs returning `200 OK` with product names, descriptions, and prices. Some IDs returned `404`, indicating missing or unavailable product objects.

## Security interpretation

This proves product-ID enumeration and demonstrates efficient Burp Intruder/Grep Extract workflow.

It does **not** prove authorization bypass by itself because product catalog data is normally public in an online shop. To classify this as a security finding, we would need stronger evidence that the endpoint exposes data that the current user should not access, such as:

- Unreleased or admin-only products
- Hidden products not discoverable through normal catalog behavior
- Wholesale/supplier-only pricing
- Internal inventory fields
- Sensitive metadata not needed by the public product page
- Access to deleted/private product records

Until then, document it as:

```text
INFO-005 Product API Enumeration Baseline
```

## Evidence

```text
24-product-id-repeater-response.png
25-authz-intruder-results.png
```

The filename `25-authz-intruder-results.png` preserves the operator's working name, but the documentation keeps the claim bounded: this is an authorization-test baseline, not a confirmed authorization finding.

## Redaction requirements

Authenticated requests may contain session material. Redact:

- `Authorization: Bearer ...`
- `token=` cookies
- JWTs
- `continueCode` or session-like values
- User email or account identifiers

Product names, prices, response status codes, and loopback hostnames are safe to retain in this local portfolio lab.

## Interview explanation

> I used the same Repeater-to-Intruder workflow against a product-detail endpoint to enumerate product IDs and extract names, descriptions, and prices into the Intruder results table. I treated the result as an informational product-catalog baseline rather than an authorization vulnerability because public product data is expected in an e-commerce application unless hidden or restricted fields are proven.
