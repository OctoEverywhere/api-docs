---
title: Set Notification Webhook API
description: Set or replace an app-owned notification webhook callback URL and event filter for the printer behind an App Connection.
og_title: Add App-Owned 3D Printer Notification Webhooks
og_description: Configure OctoEverywhere to send your app print started, completed, failed, Gadget, and error events through a callback URL you control.
authors:
    - Quinn Damerell
date: 2026-06-15
---

# Set Notification Webhook API

The Set Notification Webhook API lets an app configure a webhook callback URL for the printer associated with an App Connection.

The callback receives the same JSON payload as OctoEverywhere's user-configured [Webhook Notifications](../../webhook-notifications/overview.md). This lets your app receive print started, print complete, Gadget warning, printer error, and other notification events without asking the user to configure a webhook manually.

## HTTP Request

```{.http .apirequest title="HTTP Request"}
POST https://octoeverywhere.com/api/notifications/setfromappconnection
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
    "CallbackUrl": "https://app.example.com/octoeverywhere/webhook?tenantId=abc123&printerRef=p42",
    "EventTypes": [1, 2, 3, 7, 8, 9]
}
```

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `AppToken` | string | Conditional | Required if not sent as a header. |
| `CallbackUrl` | string | Yes | The HTTPS URL OctoEverywhere should call when a matching notification event fires. The URL may include query parameters with app-specific identifiers, tokens, or routing metadata. |
| `EventTypes` | int[] | No | Notification event enum values to deliver. If omitted, all notification events are delivered. See [Webhook Notification Event Types](../../webhook-notifications/event-types.md). |

## Replacing An Existing Webhook

This API can be called repeatedly. Each successful call replaces the previous webhook configuration for this App Connection.

Call this API again whenever your app needs to change the callback URL, query parameters, routing arguments, authentication token, or event filter. The newly saved callback URL and event filter are used immediately for all future matching notifications.

## Callback URL Metadata

`CallbackUrl` is stored and called exactly as provided. Your app can embed whatever GET parameters it needs for correlation or routing.

For example:

```text
https://app.example.com/octoeverywhere/webhook?tenantId=abc123&printerRef=p42&callbackToken=secret
```

When a notification fires, OctoEverywhere sends the webhook POST to that full URL, including the query parameters.

## Notification Payload

The callback request body is identical to the standard OctoEverywhere webhook notification payload.

```{.json .apiresponse title="Example Callback Body"}
{
    "PrinterId": "Id",
    "SecretKey": "Key",
    "PrintId": "Id",
    "EventType": 1,
    "PrinterName": "Ender3",
    "SnapshotUrl": "https://...",
    "QuickViewUrl": "https://...",
    "FileName": "MyCoolPrint",
    "DurationSec": 50,
    "Progress": 21,
    "TimeRemainingSec": 605,
    "ZOffsetMM": 300
}
```

See [Webhook JSON Payload Format](../../webhook-notifications/json-payload-format.md) for the complete payload definition.

## Callback Response Requirements

Your callback endpoint must return HTTP `200 OK` for each webhook POST. The response body is ignored, but the status code matters.

A failed delivery is any non-`200` response, including `500`, or any request that cannot reach your callback URL. After more than 5 consecutive failed delivery attempts, OctoEverywhere stops calling that URL. To resume webhook delivery after this happens, call this API again with a working callback URL.

## Event Filtering

Use `EventTypes` when your app only wants a subset of notifications.

```{.json .apirequest title="Only Completion And Failure Events"}
{
    "CallbackUrl": "https://app.example.com/octoeverywhere/webhook?printerRef=p42",
    "EventTypes": [2, 3, 7, 8, 9]
}
```

If `EventTypes` is omitted, OctoEverywhere sends all notification events to the callback URL.

The enum values are shared with the user-configured webhook notification system:

| Value | Event |
| :---: | ----- |
| `1` | Print Started |
| `2` | Print Complete |
| `3` | Print Failed |
| `4` | Print Paused |
| `5` | Print Resumed |
| `6` | Print Progress |
| `7` | Gadget Possible Failure Warning |
| `8` | Gadget Paused Print Due to Failure |
| `9` | Error |
| `10` | First Layer Complete |
| `11` | Filament Change Required |
| `12` | User Interaction Required |
| `13` | Non-Supporter Notification Limit |
| `14` | Third Layer Complete |
| `15` | Bed Cooldown Complete |
| `16` | Test Notification |

See [Webhook Notification Event Types](../../webhook-notifications/event-types.md) for event details.

## Successful Response

The successful response immediately returns the saved webhook configuration, including the callback URL and event filter that will be used for future matching notifications.

```{.json .apiresponse title="Example Response"}
{
    "Success": true,
    "CallbackUrl": "https://app.example.com/octoeverywhere/webhook?tenantId=abc123&printerRef=p42",
    "EventTypes": [1, 2, 3, 7, 8, 9]
}
```

| Name | Type | Description |
| ---- | :--: | ----------- |
| `Success` | bool | `true` when the callback configuration was saved. |
| `CallbackUrl` | string | The callback URL that was saved and will be used immediately for future matching notifications. |
| `EventTypes` | int[] | The notification event enum values that were saved. If omitted from the request, all notification events are delivered. |

## Error Responses

| Code | Meaning |
| :--: | ------- |
| `400` | Invalid input parameters, such as a missing `CallbackUrl` or invalid `EventTypes` value. |
| `500` | Internal server error. |
| `603` | App Connection not found. This App API token will never be valid again. |
| `604` | App Connection expired or revoked. This App API token will never be valid again. |

See [OctoEverywhere Custom Error Codes](../../error-codes.md) for shared handling guidance.
