---
title: App Connection Notification Webhook API
description: Add push notifications and live activity support to your 3D printing app for any 3D printer. No complex setup, no hassle.
og_title: Add 3D Printer Notification Webhooks To Your App
og_description: Add push notifications and live activity support to your 3D printing app for any 3D printer. No complex setup, no hassle.
authors:
    - Quinn Damerell
date: 2026-06-15
---

# Set Notification Webhook

App Connection Notification Webhooks allow app developers to receive real-time 3D printer notifications at any public internet URL. Each webhook callback provides all the information needed to build rich, powerful real-time push notifications, such as print status, print progress, layer stats, a live snapshot, and more!

App developers can easily set up a Google Firebase endpoint, Azure function, AWS Lambda, or similar free service handler to receive the OctoEverywhere notification and fire push notifications for your app.

!!! tip
    App Connection Webhooks are a great way to power live features like Apple's Live Activities, since they can fire at fixed time intervals!


## How It Works

### Event Types

The OctoEverywhere App Connection webhook notification will be triggered for a variety of 3D printer status updates, print progress updates, errors, and other notifications. The App Connection webhook shares a common set of events with the notifications webhook callbacks.

[See All Webhook Event Types](../../webhook-notifications/event-types.md){ .md-button .md-button--primary }

### Event JSON Payload

The `WebhookUrl` callback request body is identical to the standard OctoEverywhere webhook notification payload.

```{.json .apiresponse title="Webhook JSON Payload"}
{
    "PrinterId": "Id",
    "PrintId": "Id",
    "EventType": 1,
    ...
}
```

[See The Full Webhook JSON Payload Format](../../webhook-notifications/json-payload-format.md){ .md-button .md-button--primary }

### Lifetime & Updates

Your webhook will remain active for 180 days (~6 months) after a successful call to the `SetNotificationWebhook` API. Your app **must** call the `SetNotificationWebhook` at least once every 180 days to keep the webhook call active.

Each call to `SetNotificationWebhook` will overwrite the API parameters set by previous calls. The new values will be applied immediately.

### URL Metadata

`WebhookUrl` is stored and called exactly as provided. Your app can embed whatever GET parameters it needs for correlation or routing.

For example:

```text
https://app.example.com/octoeverywhere/webhook?tenantId=abc123&printerRef=p42&callbackToken=secret
```

When a notification fires, OctoEverywhere sends a `POST` request to the full `WebhookUrl`, including the query parameters.

### Event Type Filtering

**Optional** - App Connection developers can filter which event types they want to receive callbacks to their `WebhookUrl` for.

!!! tip
    By default, if `InclusiveEventTypesFilter` is empty or not set, all events will be fired to the `WebhookUrl`.

To filter Event Types, simply list an inclusive list of events you want to receive callbacks for using the `InclusiveEventTypesFilter` POST parameter. This value is set or updated by each call to `SetNotificationWebhook`.

## SetNotificationWebhook API

```{.http .apirequest title="HTTP Request"}
POST https://octoeverywhere.com/api/appconnection/SetNotificationWebhook
```

### Authentication

Pass the App API token either as an `AppToken` header or in the JSON request body.

| Name | Location | Type | Required | Description |
| ---- | -------- | :--: | :------: | ----------- |
| `AppToken` | Header or body | string | Yes | The `appApiToken` returned by the App Connection Portal. |

### JSON Request Body

```{.json .apirequest title="Example Request Body"}
{
    "AppToken": "<appApiToken>",
    "WebhookUrl": "https://app.example.com/octoeverywhere/webhook?tenantId=abc123&printerRef=p42",
    "InclusiveEventTypesFilter": [1, 2, 3, 7, 8, 9]
}
```

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `AppToken` | string | Conditional | Required if not sent as a header. |
| `WebhookUrl` | string | Yes | The HTTPS URL OctoEverywhere should call when a matching notification event fires. The URL may include query parameters with app-specific identifiers, tokens, or routing metadata. [Learn more](#url-metadata) |
| `InclusiveEventTypesFilter` | int[] | No | Notification event enum values to deliver. If omitted, all notification events are delivered. [Learn more](#event-type-filtering) |


### Response

The API will return a 200 `OK` if the API set was successful. Otherwise it will return an HTTP error with a JSON body payload with error details.

See [OctoEverywhere Custom Error Codes](../../error-codes.md) for shared handling guidance.


## TestWebhook API

App developers can easily invoke test webhook calls from the OctoEverywhere service to the configured `WebhookUrl` using this test API.

Note that the test webhook call will always be made, even if `InclusiveEventTypesFilter` is set to exclude it.

```{.http .apirequest title="HTTP Request"}
GET https://octoeverywhere.com/api/appconnection/TestWebhook
```

### Authentication

Pass the App API token either as an `AppToken` header..

| Name | Location | Type | Required | Description |
| ---- | -------- | :--: | :------: | ----------- |
| `AppToken` | Header | string | Yes | The `appApiToken` returned by the App Connection Portal. |

### Response

The GET API will return a JSON body indicating the HTTP status code the OctoEverywhere service received from calling the `WebhookUrl`.