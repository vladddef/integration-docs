# `GET https://t.trklinkx.com/click`

Server-to-server endpoint for submitting a lead without the landing form. It validates the lead, checks it for duplicates and registers it, then answers with JSON.

- **URL:** `https://t.trklinkx.com/click`
- **Method:** `GET`, parameters in the query string
- **Auth:** `sub30` query parameter (your API key)
- **Always HTTP `200`.** Success and failure are told apart by the `status` field in the body. Never rely on the HTTP code.
- **Content-Type:** `application/json`
- **Redirects:** `https://t.trklinkx.com/click` redirects the request to our endpoint, which returns the JSON response. You **must follow redirects** (for example `curl -L`); otherwise you will not receive the response and the lead will not be registered.

## Request

```
GET https://t.trklinkx.com/click?pid=11&offer_id=12716&sub1=6ade9d6a325c9f0001b40fdc&sub9=Ion%20Popescu&sub12=%2B40751382418&sub30=KEY&ip=203.0.113.7
```

Your HTTP client must follow the redirect to our endpoint. Without it you only get the redirect response, not the JSON result.

### Required parameters

A parameter that is missing, blank or repeated (`?pid=1&pid=2`) counts as missing.

| Parameter | Description |
|---|---|
| `sub30` | Active API key issued to you. |
| `pid` | Publisher / traffic source ID. |
| `offer_id` | Offer ID. Also part of the duplicate key. |
| `sub9` | Customer name. At least 2 characters and at least one letter (Latin or Cyrillic). |
| `sub12` | Customer phone. Any format, but 7–15 digits once non-digits are stripped. |
| `sub1` | Click ID of the visit. |

### Optional parameters

| Parameter | Description |
|---|---|
| `ip` | Customer IP. If omitted, the IP from the request headers is used (`X-Real-IP`, then the last `X-Forwarded-For` entry). |

Any other parameter is accepted and written to the log.

## Response

```json
{ "status": "ok" | "error", "data": { ... } }
```

## Behavior notes

- **Order of checks:** IP blacklist → `sub30` → required parameters → phone and name format → duplicate check → registration.
- **Duplicates:** the key is the offer plus the phone with all non-digits removed, so `+40 751 382 418` and `40751382418` match. Within 14 days of a successful submission, the same offer + phone is reported as `duplicate`.
- **Logging:** every call is logged with the API key replaced by `[redacted]`. The log still contains the customer's name and phone.
- **Retries:** only `connection_timeout` and `db` can be temporary. For those, wait and retry. A retry after a real success returns `duplicate`.
