---
title: Using App Connections
description: Call a user's printer through an App Connection URL with the same HTTP, WebSocket, file, and webcam behavior your app expects.
og_title: Use App Connections for Full Remote Printer Access
og_description: Relay HTTP, WebSocket, file, and webcam requests through OctoEverywhere while preserving methods, headers, bodies, and printer responses.
authors:
    - Quinn Damerell
date: 2026-06-15
---

# Using App Connections

After the [App Connection Portal](portal.md) creates an App Connection, your app can use the returned URL to communicate with the user's printer from anywhere.

The App Connection URL stands in for the local printer URL. For example, a local OctoPrint request might use:

```{.http .apirequest title="Local Request"}
GET http://192.168.1.5/api/login
```

The same request through an App Connection would use the returned App Connection host:

```{.http .apirequest title="App Connection Request"}
GET https://app-theverylongappconnectionid.octoeverywhere.com/api/login
```

Always use `https` for App Connection requests.

## What Gets Relayed

OctoEverywhere relays the request to the user's printer and returns the printer response. The relay preserves the HTTP method, headers, request body, response headers, response body, and response status code.

The App Connection relay supports:

- Standard HTTP API calls.
- File uploads and downloads.
- Webcam streaming.
- WebSocket connections.

For most apps, the main change is replacing the local printer host with the App Connection host when remote access is needed.

## Authentication

If the App Connection uses authentication, your app must include the selected authentication header on every request. This includes normal HTTP requests, WebSocket handshakes, file transfers, and webcam streams.

For HTTP Basic auth:

```http
Authorization: Basic <base64(username:password)>
```

For bearer auth:

```http
Authorization: Bearer <authBearerToken>
```

The portal returns the generated auth values on the completion URL. Store them securely because they are only returned once.

## Local And Remote Connection Strategy

If your app can reach the printer locally, use the local connection for the best LAN experience. When the local printer is not reachable, use the App Connection URL.

The portal may return `printerLastKnownLocalIp`, and the [App Connection Info API](apis/info-api.md) can return `PrinterLocalIp`. These values can help your app discover a local printer after it has been set up through OctoEverywhere.

## Special Considerations

Your app should handle these OctoEverywhere-specific cases:

- Include the selected App Connection auth header on every request.
- Detect shared OctoEverywhere `6xx` error codes and show user-facing recovery UI.
- Show clear states for common problems like printer offline, App Connection revoked, App Connection expired, or supporter status missing.
- Switch between local printer access and App Connection access when possible.
- Respect account limits for file upload size, file download size, and webcam streaming. Query current limits with the [App Connection Info API](apis/info-api.md).

## Error Codes

OctoEverywhere uses shared `6xx` HTTP status codes when the relay or App APIs need to report an OctoEverywhere-specific error without colliding with status codes returned by the user's printer.

See [OctoEverywhere Custom Error Codes](../error-codes.md) for the full list.

!!! tip
    If a request returns `605`, the user's account no longer has Supporter Perks. Link them to `https://octoeverywhere.com/appportal/v1/nosupporterperks?appid=<your_app_id>` so OctoEverywhere can explain the issue and recovery path.
