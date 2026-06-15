---
title: App Connection Info API
description: Get current state and limits for an App Connection.
authors:
    - Quinn Damerell
date: 2026-06-15
---

# App Connection Info API

The App Connection Info API returns current state and useful details for one App Connection.

Use it to:

- Determine whether an App Connection is still valid.
- Check whether the printer is currently connected to OctoEverywhere.
- Get the printer name and last known local IP address.
- Get the user-facing OctoEverywhere printer URL.
- Read current account limits for file transfers and webcam streaming.

## HTTP Request

```{.http .apirequest title="HTTP Request"}
GET https://octoeverywhere.com/api/appconnection/info
```

## Headers

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `AppToken` | string | Yes | The `appApiToken` returned by the App Connection Portal. |

## Successful Response

```{.json .apiresponse title="Example 200 Response"}
{
    "PrinterName": "My Printer",
    "PrinterLocalIp": "192.168.1.5",
    "IsOnline": true,
    "LastDisconnectTimeUtc": "2026-06-15T01:20:30Z",
    "LastConnectionTimeUtc": "2026-06-15T01:25:30Z",
    "UserFacingPrinterUrl": "https://octoeverywhere.com/view/My%20Printer",
    "PrinterLimits": {
        "MaxDownloadFileSizeBytes": 104857600,
        "MaxUploadFileSizeBytes": 104857600,
        "MaxSingleWebcamStreamLengthSeconds": 600
    }
}
```

| Name | Type | Description |
| ---- | :--: | ----------- |
| `PrinterName` | string | The name assigned to the printer in OctoEverywhere by the user. |
| `PrinterLocalIp` | string | Last known LAN IP address for the printer, if known. Useful for local printer discovery after OctoEverywhere setup. |
| `IsOnline` | bool | `true` when the printer is currently connected to OctoEverywhere. |
| `LastDisconnectTimeUtc` | string | Last time the printer disconnected from OctoEverywhere. |
| `LastConnectionTimeUtc` | string | Last time the printer connected to OctoEverywhere. |
| `UserFacingPrinterUrl` | string | URL the user can open for normal OctoEverywhere remote access. This URL requires the user to be logged in and is useful for app web views or user-facing links. It is not the app-authenticated App Connection URL. |
| `PrinterLimits` | object | Current account limits that can affect App Connection requests. |

### PrinterLimits

| Name | Type | Description |
| ---- | :--: | ----------- |
| `MaxDownloadFileSizeBytes` | int | Maximum file size the user can download through the App Connection. Depends on the user's supporter level. |
| `MaxUploadFileSizeBytes` | int | Maximum file size the user can upload through the App Connection. Depends on the user's supporter level. |
| `MaxSingleWebcamStreamLengthSeconds` | int | Maximum duration for one webcam stream session. |

## Determine App Connection State

Use the HTTP status code and `IsOnline` together:

| Result | Meaning |
| ------ | ------- |
| `200` and `IsOnline` is `true` | The App Connection is valid and the printer is connected to OctoEverywhere. |
| `200` and `IsOnline` is `false` | The App Connection is valid, but the printer is not currently connected to OctoEverywhere. |
| `603` or `604` | The App Connection is no longer valid and must be recreated. |
| `605` | The App Connection is valid, but the owner account does not currently have Supporter Perks. |

## Error Responses

| Code | Meaning |
| :--: | ------- |
| `400` | No `AppToken` was found. |
| `600` | Temporary server, plugin, or unknown error. |
| `601` | Printer is not connected to OctoEverywhere. |
| `602` | OctoEverywhere's connection to the printer timed out. |
| `603` | App Connection not found for the given token. |
| `604` | App Connection revoked or expired. |
| `605` | App Connection owner's account is no longer a supporter. |

See [OctoEverywhere Custom Error Codes](../../error-codes.md) for shared handling guidance.
