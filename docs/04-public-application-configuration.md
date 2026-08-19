# Public Application Configuration Analysis

## Status

Testing completed by the operator; screenshot and response-content review pending before classification as a finding.

## Observed workflow

1. Continued unauthenticated application mapping in Burp's embedded Chromium browser.
2. Opened the Photo Wall.
3. Identified the following request in Proxy HTTP history:

```http
GET /rest/admin/application-configuration HTTP/1.1
Host: 127.0.0.1:3000
```

4. Sent the request to Repeater.
5. Removed the conditional-cache header:

```http
If-None-Match: ...
```

6. Sent the modified request and received a full response:

```http
HTTP/1.1 200 OK
```

## Header interpretation

Removing `If-None-Match` is relevant because it prevents the request from asking whether the cached ETag is still valid. This can cause the server to return the complete representation instead of `304 Not Modified` with no body.

Removing:

```http
Connection: keep-alive
```

is not necessary for obtaining the response body. That header controls whether the TCP connection may be reused; it is not a cache validator.

## Classification guardrail

The endpoint is publicly accessible to the application's own frontend, so exposure of configuration data is not automatically a vulnerability. Classify it only after reviewing the actual fields.

### Potentially reportable content

- Secrets, API keys, or credentials
- Internal-only hostnames or administrative URLs
- Personal emails or identifiers
- Debug settings or hidden administrative capabilities
- Security-control configuration that materially aids exploitation

### Likely informational or expected content

- Application name and theme
- Logo/image paths
- Public social-media links
- Public feature flags required by the frontend
- Public privacy/contact links

If the response contains only frontend-required public settings, record it as an application-mapping observation rather than a vulnerability.

## Suggested Burp tab name

Until the body is reviewed:

```text
INFO-001 Application Configuration
```

If the response contains confirmed reportable information and the lab convention marks findings with a star:

```text
* INFO-001 Application Configuration Exposure
```

## Evidence marker

```text
15-burp-application-configuration-response.png
```

Capture the Repeater request and enough of the `200 OK` JSON response to show the endpoint and representative fields. Before publishing, redact any secrets, tokens, real emails, public IPs, external account identifiers, or private infrastructure names.

## Interview explanation

> During unauthenticated mapping, I identified a public application-configuration endpoint. The browser initially relied on ETag cache validation, so I sent the request to Repeater and removed the `If-None-Match` header to retrieve a full `200 OK` representation. I then reviewed the returned fields to distinguish expected frontend configuration from potentially sensitive information rather than labeling every public configuration endpoint as a vulnerability.
