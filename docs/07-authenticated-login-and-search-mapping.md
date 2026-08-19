# Authenticated Login and Search Mapping

## Status

Login token exposure confirmed; search request retained as an informational baseline for later testing.

## Login workflow

After creating a lab user and logging in, Burp Proxy captured:

```http
POST /rest/user/login HTTP/1.1
Host: 127.0.0.1:3000
```

The response returned:

```http
HTTP/1.1 200 OK
```

and included an authentication token.

Returning a token after login is expected. The important security observation is that the token could be decoded in Burp Decoder and contained a sensitive user object, including a password-hash field and additional account/security metadata.

See finding:

```text
F-004 — Sensitive User Claims Exposed in Client-Readable JWT
```

## Decoder workflow

The token was copied into Burp Decoder. Decoding showed JWT-style structure:

```text
header.payload.signature
```

JWT header and payload sections are Base64URL-encoded. They are not encrypted by default. The signature protects integrity, not confidentiality.

This is why client-readable tokens should contain only minimum session claims.

## Search request mapping

A search request was captured and sent to Repeater for later analysis:

```http
GET /rest/products/search?q= HTTP/1.1
Host: 127.0.0.1:3000
```

The response returned:

```http
HTTP/1.1 200 OK
```

with product JSON.

## Header note

Removing `If-None-Match` is useful when a request returns `304 Not Modified` and the goal is to retrieve a fresh response body.

Removing:

```http
Connection: keep-alive
```

is not needed for cache-bypass testing. It affects connection reuse, not whether the server returns a cached validation response.

## Classification for search

The search request is currently:

```text
Informational baseline / later input-validation testing target
```

It is not a vulnerability yet. Future testing can evaluate:

- Search behavior with ordinary terms
- Special characters and encoding
- Error handling
- Reflected content
- SQL/NoSQL injection lessons if appropriate for the Juice Shop challenge path

## Evidence markers

```text
19-user-login-token-shown.png
20-decoded-base64-password-hash-finding.png
21-searchbar-repeater-tab.png
```

## Suggested Burp tab names

```text
* F-004 JWT Sensitive Claims
INFO-004 Search Baseline
```

The search tab should remain informational unless later testing confirms a specific weakness.
