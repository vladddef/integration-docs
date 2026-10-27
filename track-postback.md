# `GET /landing/track-postback`

Server-to-server endpoint for submitting a lead without the landing form. It validates the lead, checks it for duplicates and registers it, then answers with JSON.

- **URL:** `/landing/track-postback`
- **Method:** `GET`, parameters in the query string
- **Auth:** `api_key` query parameter
- **Always HTTP `200`.** Success and failure are told apart by the `status` field in the body. Never rely on the HTTP code.
- **Content-Type:** `application/json`

## Request

```
GET /landing/track-postback?api_key=KEY&pid=5345&offer_id=13425&cpa_offer_id=3920&sub9=Ion%20Popescu&sub12=%2B40751382418&sub1=6ade9d6a325c9f0001b40fdc&ip=203.0.113.7
```

### Required parameters

A parameter that is missing, blank or repeated (`?pid=1&pid=2`) counts as missing.

| Parameter | Description |
|---|---|
| `api_key` | Active API key issued to you. |
| `pid` | Publisher / traffic source ID. |
| `offer_id` | Offer ID on the traffic-source side. |
| `cpa_offer_id` | Offer ID on the CPA side. Also part of the duplicate key. |
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

- **Order of checks:** IP blacklist → `api_key` → required parameters → phone and name format → duplicate check → registration.
- **Duplicates:** the key is `cpa_offer_id` plus the phone with all non-digits removed, so `+40 751 382 418` and `40751382418` match. Within 14 days of a successful submission, the same offer + phone is reported as `duplicate`.
- **Logging:** every call is logged as `track-postback query: {...}` with `api_key` replaced by `[redacted]`. The log still contains the customer's name and phone.
- **Retries:** only `upstream` and `db` can be temporary. For those, wait and retry. A retry after a real success returns `duplicate`.
