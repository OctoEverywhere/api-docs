# Plugin APIs

## Overview

OctoEverywhere's Plugin APIs are a set of 3D printer platform-agnostic APIs that enable common access, monitoring, and control of your 3D printer, working with any 3D printer supported by OctoEverywhere. These Plugin APIs are callable anywhere from the public internet and are secured by OctoEverywhere's advanced access control system.

### Supported Printer Platforms And 3D Printer Makers

- OctoPrint
- Klipper (Moonraker)
- Bambu Lab
- Prusa
- Elegoo
- Sovol
- Snapmaker
- And more!

### Plugin APIs Include

- Real-time printer status, print status, AI failure detection status, and more.
- Live multi-camera webcam snapshots and webcam streams.
- File list, upload, download, and delete.
- Printer control, including pause, cancel, home, move axis, extrude, and more.
- One-shot commands and responses to the 3D printers control the WebSocket or MQTT server.
- Full MQTT over WebSocket real-time MQTT remote access proxy.

## Get Started

### Setup OctoEverywhere

You must have the OctoEverywhere plugin set up and connected to your 3D printer. For most 3D printers, the OctoEverywhere plugin can run directly on your printer.

[Set Up The OctoEverywhere Plugin](https://octoeverywhere.com/getstarted?source=docs_plugin_api_index){ .md-button .md-button--primary }

### Get Your Secure Remote Access URL

The Plugin APIs can be called using a [Shared Connection](https://octoeverywhere.com/sharedconnections?source=docs_plugin_api) or [App Connection](../app-connections/index.md) URL. The common calling format is:

`https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/...`

Where `<unique_id>` is your unique shared connection or app connection subdomain.

### Authentication

The Plugin APIs are callable from anywhere on the public internet and are secured using OctoEverywhere's advanced access control system. The Plugin APIs share the authentication required by the [Shared Connection](https://octoeverywhere.com/sharedconnections?source=docs_plugin_api) or [App Connection](../app-connections/index.md).

### HTTP Error Codes

Plugin APIs return common [OctoEverywhere Error Codes](../error-codes.md) for OctoEverywhere service-side issues and standard HTTP error codes for plugin API issues.

## Plugin APIs

### [Printer Status](printer-status.md)

A printer-agnostic JSON object with current printer status, print progress, percentage complete, estimated time remaining, and more.

### [Printer Control](printer-control.md)

Printer-agnostic control for pausing or canceling prints, toggling lights, homing axes, extruding filament, and more.

### [Webcam APIs](webcam-api.md)

A set of printer-agnostic webcam APIs, including camera lists, multi-camera snapshots, and multi-camera streams.

### [File APIs](files-api.md)

A set of printer-agnostic files APIs, including list, upload, download, and delete.

### [MQTT over WebSocket Proxy](mqtt-websocket-proxy.md)

**Works With:** Bambu Lab & Elegoo Centauri Carbon 2

Bambu Lab and Elegoo Centauri Carbon 2 printers use MQTT for low-level status, control, and event streams. OctoEverywhere provides a secure, internet-facing, standards-compliant MQTT-over-WebSocket proxy, allowing apps to use standard MQTT client libraries to connect to the printer from anywhere.

The original JSON-based WebSocket transport is still available for existing integrations, but new clients should use the standards-compliant proxy.

[Legacy JSON Transport Reference](mqtt-websocket-proxy-json-legacy.md)
