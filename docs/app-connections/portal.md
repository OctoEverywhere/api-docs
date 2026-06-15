---
title: App Connection Portal
description: Use OctoEverywhere's hosted portal to let users authorize an App Connection, choose a printer, and return secure app access tokens.
og_title: Launch App Connections with the Hosted Portal
og_description: Guide users through a secure OctoEverywhere authorization flow and receive the App Connection tokens your integration needs.
authors:
    - Quinn Damerell
date: 2026-06-15
---

# App Connection Portal

The App Connection Portal is the hosted setup flow your app uses to create an App Connection. It is designed to be easy for users, complete for app developers, and robust across account, printer, and supporter-status scenarios.

[Try The Demo Portal](https://octoeverywhere.com/appportal/v1/?appid=devtest){ .md-button .md-button--primary }

!!! note
    The portal experience is optimized for mobile and tablet layouts, which makes it suitable for in-app web views.

## User Flow

1. A user chooses to set up remote printer access in your app.
2. Your app opens the OctoEverywhere App Connection Portal in an in-app web view.
3. The portal shows a welcome screen.
4. If the user does not have an OctoEverywhere account, the portal helps them create one.
5. If the user does not have a printer connected to OctoEverywhere, the portal walks them through the setup.
6. If the user does not have Supporter Perks, the portal explains why App Connections currently require supporter status and helps them upgrade if they choose to.
7. The user selects a printer and authorizes your app.
8. The portal navigates to the completion URL.
9. Your app reads the returned query parameters and stores the App Connection context.

## App Integration Flow

1. Open an in-app web view to the portal start URL.
2. Include the required and optional query parameters for your app.
3. Watch web view navigation until the portal reaches the completion URL or your custom `returnUrl`.
4. Parse and store the returned query parameters.
5. Use the returned App Connection URL, credentials, and App API token for future printer access.

!!! warning
    `appApiToken`, `octoPrintApiKey`, and the returned `auth*` values are only returned once. They cannot be queried from OctoEverywhere later. If they are lost, the user must create a new App Connection.

## Start The Portal

```{.http .apirequest title="HTTP Request"}
GET https://octoeverywhere.com/appportal/v1
```

Example:

```text
https://octoeverywhere.com/appportal/v1/?appid=devtest
```

### Query Parameters

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `appId` | string | Yes | The App ID assigned to your app by OctoEverywhere. It identifies which app created the App Connection and is used in the portal UI. |
| `returnUrl` | string | No | URL-encoded completion URL. If omitted, the portal navigates to `https://octoeverywhere.com/appportal/v1/complete`. Returned query parameters are appended to this URL. |
| `printerId` | string | No | The OctoEverywhere Printer ID the user is trying to connect to. If it belongs to the user's account, the portal selects it by default. If not, the portal starts the flow to add it. |
| `appLogoUrl` | string | No | URL-encoded logo URL shown in the portal. The image is displayed in a 100x100px square. Logos with solid circular backgrounds and transparent corners work best. |
| `octoPrintApiKeyAppName` | string | No | URL-encoded app name to show in OctoPrint's API key settings. If present, the portal attempts to generate an OctoPrint API key for your app. See [OctoPrint API Key Generation](octoprint-api-key-generation.md). |

### About `authType`

Older portal docs mentioned an `authType` parameter. App Connections now require secure header-based authentication for remote apps, so the value is effectively `enhanced`. New integrations do not need to set `authType`.

## Finding A Printer ID Locally

If your app already has local access to the OctoPrint instance, it can query the OctoEverywhere plugin for the OctoEverywhere Printer ID before starting the portal.

```{.http .apirequest title="HTTP Request"}
GET http://<local-octoprint-ip>/api/plugin/octoeverywhere
```

Pass that Printer ID as `printerId` to make the portal select the intended printer by default.

## Handle Portal Completion

When setup succeeds or fails, the portal navigates to either the default completion URL or your custom `returnUrl`.

```{.http .apiresponse title="Default Completion URL"}
GET https://octoeverywhere.com/appportal/v1/complete
```

Your app should monitor the web view navigation, detect the completion URL, parse the query parameters, store the App Connection values, and close the web view.

### Completion Parameters

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `success` | bool | Yes | Indicates whether the portal flow succeeded. |
| `id` | string | On success | The new App Connection ID. Store it with the user's printer record. |
| `url` | string | On success | The App Connection root URL. Use this in place of the local printer URL for remote access. |
| `appApiToken` | string | On success | API token for OctoEverywhere App APIs for this App Connection. Store it securely; it is only returned once. |
| `printername` | string | On success | The user-assigned printer name. |
| `userPrinterAccessUrl` | string | On success | User-facing OctoEverywhere printer URL. This is useful for UI links, but it is not the app-authenticated App Connection URL. |
| `printerLastKnownLocalIp` | string | No | Last known LAN IP address for the printer, if known. This can help your app discover and prefer the local printer connection when available. |
| `authBasicHttpUser` | string | Conditional | Generated HTTP Basic auth username. Returned when basic auth is used. |
| `authBasicHttpPassword` | string | Conditional | Generated HTTP Basic auth password. Returned when basic auth is used. |
| `authBearerToken` | string | Conditional | Generated bearer token. Returned when bearer auth is used. |
| `octoPrintApiKey` | string | No | Generated OctoPrint API key, if `octoPrintApiKeyAppName` was requested and key creation succeeded. |

Use the exact query keys returned by the portal. Some legacy docs showed lowercase variants for the basic-auth keys, so tolerant parsing can help older app builds.
