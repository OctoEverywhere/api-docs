---
title: Gadget Status API
description: Read Gadget AI failure detection status for an App Connection.
authors:
    - Quinn Damerell
date: 2026-06-15
---

# Gadget Status API

The Gadget Status API returns the current state of [Gadget, OctoEverywhere's AI failure detection assistant](https://octoeverywhere.com/Gadget), for the printer associated with an App Connection.

Gadget is free and enabled by default for the OctoEverywhere community. Adding this API to your app lets users see AI failure detection state, status, color, and rating directly in your UI.

## HTTP Request

```{.http .apirequest title="HTTP Request"}
POST https://octoeverywhere.com/api/gadget/GetStatusFromAppConnection
```

## Authentication

Pass the App API token either as an `AppToken` header or in the JSON request body.

| Name | Location | Type | Required | Description |
| ---- | -------- | :--: | :------: | ----------- |
| `AppToken` | Header or body | string | Yes | The `appApiToken` returned by the App Connection Portal. |

## JSON Request Body

```{.json .apirequest title="Example Request Body"}
{
    "AppToken": "<appApiToken>"
}
```

`AppToken` is only required in the body if it was not sent as a header.

## Polling Guidance

Gadget's actual inspection rate changes dynamically based on server load. This API is designed to be polled while the user can see the status and the printer is printing.

Recommended polling behavior:

- If the user can see the status and the printer is printing, poll every `10` seconds. This is the maximum rate allowed per App Connection.
- If the user cannot see the status but the printer is printing, pause polling or significantly reduce the rate until the user can see it again.
- If the printer is not printing, do not poll because the result will not change.
- If Gadget is disabled for the account, occasional low-frequency polling is acceptable to detect a settings change.
- If Gadget is disabled for the current print, wait until a new print starts before polling again.

## Successful Response

```{.json .apiresponse title="Example 200 Response"}
{
    "State": 3,
    "Status": "Looking Good",
    "StatusColor": "g",
    "Rating": 8
}
```

| Name | Type | Description |
| ---- | :--: | ----------- |
| `State` | int | Gadget state enum. See [Gadget States](#gadget-states). |
| `Status` | string | Short user-facing status text for the active print. Expected to be about four words or fewer. |
| `StatusColor` | string | One-character status color: `g` green, `y` yellow, `r` red, or `w` white/neutral. |
| `Rating` | int | Print-quality rating from `1` to `10`. See [Rating Values](#rating-values). |

`Status` is only meaningful when `State` is `3` (`Active`). For other states, there may be no active print status to show.

## Gadget States

| Value | State | Meaning |
| :---: | ----- | ------- |
| `0` | `Disabled` | Gadget is disabled on the user's account for all printers. |
| `1` | `DisabledForCurrentPrint` | Gadget is disabled for this printer for the rest of the current print. |
| `2` | `Idle` | Gadget is not currently monitoring a print. |
| `3` | `Active` | Gadget is monitoring the active print. |
| `4` | `PausedDueToFailure` | The print is paused because Gadget detected a failure. |

## Rating Values

Use `Status` and `StatusColor` when possible so your app automatically reflects future Gadget wording changes. If you need your own display logic, use `Rating`.

| Value | Meaning |
| :---: | ------- |
| `1` | Print failure. |
| `2` | Probable print failure. |
| `3` | Possible print failure. |
| `4` | Monitoring a possible print failure. |
| `5` | Monitoring a possible print failure. |
| `6` | Print is looking good. |
| `7` | Print is looking good. |
| `8` | Print is looking great. |
| `9` | Print is looking great. |
| `10` | Print looks perfect. |

## User Links

If the user needs Gadget information or settings, link them to:

```text
https://octoeverywhere.com/gadget
```

If the user wants the full Gadget status directly from OctoEverywhere, link them to their printer Quick View page:

```text
https://octoeverywhere.com/view/<printer-name>
```

The printer name is returned by the portal and can also be read from the [App Connection Info API](info-api.md).

## Error Responses

| Code | Meaning |
| :--: | ------- |
| `400` | Invalid required POST parameters. |
| `500` | Internal server error. |
| `601` | Printer is not connected to OctoEverywhere. |
| `602` | Connection to the printer timed out. |
| `603` | App Connection not found. This App API token will never be valid again. |
| `604` | App Connection expired or revoked. This App API token will never be valid again. |
| `610` | Plugin update required. The user must update their OctoEverywhere plugin to use Gadget. |
| `611` | No beta access. |

See [OctoEverywhere Custom Error Codes](../../error-codes.md) for shared handling guidance.
