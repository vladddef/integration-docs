# Track Postback Conversion API

## Endpoint
```
GET https://t.trklinkx.com/click
```

## Authentication

Pass your API key as the `api_key` query parameter. Keys are provisioned by the server administrator.

## Parameters

| Parameter     | Required | Description                                 |
|---------------|----------|---------------------------------------------|
| `sub30`     | yes      | Your API key                                |
| `pid`         | yes      | Partner/publisher ID                        |
| `offer_id`    | yes      | Offer ID                                    |
| `sub9`        | no       | Lead name                                   |
| `sub12`       | no       | Lead phone number                           |
| `ua`          | no       | User agent string                           |
| `ip`          | no       | User ip                                     |
| `device_type` | no       | Device type (`mobile`, `tablet`, `desktop`) |

## Responses

| Status | Meaning                                               |
|--------|-------------------------------------------------------|
| `200`  | Conversion tracked successfully                       |
| `400`  | Conversion failed |
| `401`  | Missing or invalid `api_key`                          |
