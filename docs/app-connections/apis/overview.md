---
title: App Connection APIs
description: Query App Connection state, limits, Live Links, and Gadget status.
authors:
    - Quinn Damerell
date: 2026-06-15
---

# App Connection APIs

The OctoEverywhere App APIs let your app query OctoEverywhere for information and actions related to an existing App Connection.

Use the App API token returned by the [App Connection Portal](../portal.md). This token is scoped to one App Connection and one printer.

## Base URL

```text
https://octoeverywhere.com
```

## Authentication

Most App APIs accept the portal's `appApiToken` as an `AppToken` header. Some POST APIs also accept `AppToken` in the JSON body.

```http
AppToken: <appApiToken>
```

## Available APIs

| API | Purpose |
| --- | ------- |
| [App Connection Info API](info-api.md) | Check App Connection validity, printer online state, printer details, user-facing printer URL, and account limits. |
| [Create Live Link API](live-links.md) | Create a shareable Live Link for the printer associated with an App Connection. |
| [Gadget Status API](gadget-status.md) | Read Gadget AI failure detection state, status, color, and rating for the active print. |

## App Connection State

If your app cannot connect through the App Connection URL, call the [App Connection Info API](info-api.md) to understand why. The result can tell you whether:

- The App Connection is valid and the printer is connected to OctoEverywhere.
- The App Connection is valid but the printer is not connected to OctoEverywhere.
- The App Connection is no longer valid.
- The user temporarily cannot use App Connections because their account does not have Supporter Perks.

## User Limits

The [App Connection Info API](info-api.md) also returns current limits for the user's account, including file upload/download and webcam-streaming limits. Use these values to set expectations before a request and to explain `607`, `608`, or `609` errors when a limit is reached.

## Error Codes

App APIs use the same shared OctoEverywhere error codes as the App Connection relay. See [OctoEverywhere Custom Error Codes](../../error-codes.md).
