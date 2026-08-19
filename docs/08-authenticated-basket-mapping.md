# Authenticated Basket Mapping and BOLA Testing

## Status

Confirmed basket object-level authorization weakness in the local Juice Shop lab.

## Objective

Compare authenticated basket behavior against earlier unauthenticated guest-basket behavior and test whether numeric basket IDs are authorization-protected.

Earlier unauthenticated basket behavior appeared to use browser-side guest state. After login, the application generated server-side basket routes.

## Discovery

After logging in and adding Apple Juice to the basket, the application generated a basket request with a numeric object identifier:

```http
GET /rest/basket/6 HTTP/1.1
Host: 127.0.0.1:3000
```

A numeric object ID in a route should trigger an object-authorization question:

> What happens if this ID is changed to another valid object?

## Manual Repeater test

The basket request was sent to Repeater. The basket ID was changed from `6` to `5`:

```http
GET /rest/basket/5 HTTP/1.1
```

The operator observed that the response returned contents for another basket associated with `UserId: 16`, including Eggfruit Juice and Raspberry Juice.

This behavior supports a Broken Object Level Authorization / IDOR finding.

## Intruder enumeration

To check whether this could be repeated across multiple IDs, the request was sent from Repeater to Intruder.

Configuration:

```text
Attack type: Sniper
Payload type: Numbers
From: 1
To: 10
Step: 1
URL encode payloads: off
```

Payload position:

```http
GET /rest/basket/§5§ HTTP/1.1
```

The range was intentionally small because this is a local training lab and the purpose was controlled validation, not high-volume scanning.

## Grep Extract

Burp Intruder's **Grep - Extract** feature was used to pull the owner indicator from responses:

```json
"UserId":
```

This made it possible to compare returned basket owners without manually opening every response.

## Evidence

```text
22-intruder-payload.png
23-filtered-userid-payload-results.png
```

Recommended additional screenshot:

```text
26-basket-idor-direct-repeater-response.png
```


## Classification

This is documented as:

```text
F-005 — Basket IDOR / Broken Object Level Authorization
```

A successful GET to another user's basket is a read-access authorization issue. Do not claim unauthorized modification or checkout impact unless a later test proves it.
