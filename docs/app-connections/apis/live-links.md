---
title: Create Live Link API
description: Create a shareable Live Link from an App Connection.
authors:
    - Quinn Damerell
date: 2026-06-15
---

# Create Live Link API

The Create Live Link API creates a shareable Live Link for the printer associated with an App Connection.

Live Links let users share temporary live access to a printer. When print-tracking mode is enabled, the Live Link follows the current print and becomes a static end-cap page when the print completes.

## HTTP Request

```{.http .apirequest title="HTTP Request"}
POST https://octoeverywhere.com/api/live/createfromappconnection
```

## Authentication

Pass the App API token either as an `AppToken` header or in the JSON request body.

| Name | Location | Type | Required | Description |
| ---- | -------- | :--: | :------: | ----------- |
| `AppToken` | Header or body | string | Yes | The `appApiToken` returned by the App Connection Portal. |

## JSON Request Body

```{.json .apirequest title="Example Request Body"}
{
    "AppToken": "<appApiToken>",
    "ExpirationHours": 24,
    "EndWhenCurrentPrintIsComplete": false
}
```

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `AppToken` | string | Conditional | Required if not sent as a header. |
| `ExpirationHours` | number | No | Number of hours until the Live Link expires. Must be between `1` and `17520`. Defaults to `24` hours if omitted. |
| `EndWhenCurrentPrintIsComplete` | bool | No | When `true`, `ExpirationHours` is ignored and the Live Link tracks the active print. When the print completes, live access ends and the link becomes a static print result page. |

!!! note
    The API currently does not accept Live Link metadata such as title, model URL, or filament URL.

## Successful Response

```{.json .apiresponse title="Example Response"}
{
    "LiveId": "abc123",
    "Url": "https://octoeverywhere.com/live/abc123",
    "ShortUrl": "https://oe.ink/l/abc123"
}
```

| Name | Type | Description |
| ---- | :--: | ----------- |
| `LiveId` | string | Live Link ID. This is a random alphanumeric string, and its length may change over time. |
| `Url` | string | Full URL the user can share. |
| `ShortUrl` | string | Short URL that redirects to `Url`. |

## Error Responses

| Code | Meaning |
| :--: | ------- |
| `400` | Invalid input parameters. |
| `500` | Internal server error. |
| `603` | App Connection not found. This App API token will never be valid again. |
| `604` | App Connection expired or revoked. This App API token will never be valid again. |
| `613` | No active print. Returned only when `EndWhenCurrentPrintIsComplete` is `true`, because print-tracking Live Links require a current active print. |

See [OctoEverywhere Custom Error Codes](../../error-codes.md) for shared handling guidance.
