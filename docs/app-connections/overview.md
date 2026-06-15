---
title: App Connections
description: Add OctoEverywhere remote printer access to your app.
authors:
    - Quinn Damerell
date: 2026-06-15
---

# App Connections

An App Connection is a third-party app integration with OctoEverywhere that lets your app securely access a user's printer from anywhere. After a user completes the App Connection Portal flow, your app receives a unique printer URL, authentication credentials, an App API token, and useful printer context.

For most printer communication, an App Connection acts like the user's local printer URL. Your app keeps using the printer's normal HTTP APIs, WebSockets, webcam streams, file transfers, and other routes, but replaces the local host with the App Connection host.

!!! tip
    App Connections are meant for full remote access to a user's printer. If you want a common printer-agnostic API across OctoPrint, Klipper, Bambu Lab, PrusaLink, Elegoo, and other platforms, see the [3D Printer APIs](../plugin-api/).

[Start With The App Portal](portal.md){ .md-button .md-button--primary }

## Benefits

App Connections give your app:

- A secure remote access option for existing users.
- A hosted setup flow that handles OctoEverywhere login, account setup, printer setup, printer selection, and app authorization.
- Raw access to the same printer HTTP APIs, WebSockets, webcam streams, upload/download paths, and responses your app would use locally.
- A unique App Connection hostname protected by header-based authentication.
- Optional local printer details, including the printer's last known LAN IP address.
- Optional OctoPrint API key generation during setup, so users do not need to manually find and paste an OctoPrint API key.
- OctoEverywhere APIs for App Connection state, printer online state, account limits, Live Links, Gadget AI failure detection status, and app-owned notification webhooks.
- A path to be listed by OctoEverywhere as an integrated app.

## Integration Pieces

App Connections have five useful integration points:

| Area | What It Does |
| ---- | ------------ |
| [App Connection Portal](portal.md) | Creates an App Connection through a hosted "OAuth-style" setup flow. |
| [Using App Connections](usage.md) | Explains how to call the user's printer through the App Connection URL. |
| [App APIs](apis/overview.md) | Lets your app query App Connection state, printer status, limits, Live Links, Gadget status, and app-owned notification webhooks. |
| [OctoPrint API Key Generation](octoprint-api-key-generation.md) | Optionally asks the portal to generate an OctoPrint API key for your app. |
| Local OctoEverywhere plugin API | Lets your app discover the OctoEverywhere Printer ID from a local OctoPrint instance before starting the portal. |

## App Connection Lifecycle

An App Connection is created when the portal flow succeeds. Each App Connection gives your app access to one printer for one user. If a user connects multiple printers, each printer gets its own App Connection URL, credentials, and App API token.

An App Connection remains valid until one of these events happens:

- The user deletes their OctoEverywhere account.
- The user removes the printer from their account.
- The user revokes the App Connection from the OctoEverywhere website.

Your app can query the [App Connection Info API](apis/info-api.md) at any time to check whether an App Connection is still valid and whether the printer is currently connected to OctoEverywhere.

## Supporter Status

App Connections require more OctoEverywhere service resources than normal remote access, so App Connections currently require the printer owner to have Supporter Perks.

If the user's account loses Supporter Perks, App Connection requests return the shared `605` error code. If the user becomes a supporter again later, existing App Connections become valid again without requiring setup again.

When your app receives `605`, link the user to:

```text
https://octoeverywhere.com/appportal/v1/nosupporterperks?appid=<your_app_id>
```

Using this shared page keeps the explanation current if App Connection supporter details change.

## Authentication

Every App Connection starts with a unique high-entropy subdomain. The subdomain is not treated as the only secret, because DNS names are visible during DNS resolution. App Connections also require header-based authentication sent over HTTPS.

App Connections support:

- HTTP Basic authentication.
- Bearer token authentication.

Both options are secure when sent over HTTPS. Use whichever option is easier for your app to apply consistently. Your app must send the selected authentication header on every App Connection request, including HTTP requests, WebSocket connections, and webcam streaming requests.

The portal returns the generated authentication values on the completion URL. Store them securely; they are only returned once.

## Recommended Setup Flow

### 1. Integrate The App Connection Portal

The portal is required to create App Connections. It gives users a hosted flow to log in, create an OctoEverywhere account if needed, connect a printer if needed, select a printer, and authorize your app.

[Get Started With The App Portal](portal.md){ .md-button }

### 2. Use The App Connection URL

After the portal succeeds, replace the user's local printer host with the returned App Connection URL for printer HTTP APIs, WebSockets, webcam streams, and file transfer paths.

[Use An App Connection](usage.md){ .md-button }

### 3. Query OctoEverywhere App APIs

Use the App APIs to check App Connection state, printer online state, printer details, account limits, Live Link creation, Gadget AI failure detection status, and app-owned notification webhooks.

[Explore App APIs](apis/overview.md){ .md-button }

### 4. Optional: Generate An OctoPrint API Key

If your app targets OctoPrint, the portal can optionally request an OctoPrint API key for your app and return it on the completion URL.

[Enable OctoPrint API Key Generation](octoprint-api-key-generation.md){ .md-button }

## Feedback

App Connections were built to make third-party app integration easier. If something is missing or an API would make your app better, [contact the OctoEverywhere developer team](https://octoeverywhere.com/support).
