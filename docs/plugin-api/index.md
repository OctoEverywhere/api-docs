# Plugin APIs

## Overview

The OctoEverywhere plugin exposes printer-agnostic APIs that can be accessed from any OctoEverywhere remote access connection. These APIs abstract the printer host and give you consistent access to status, control, webcams, and more.

!!! tip
    These APIs are a great way to work with status, control, webcams, and more across every 3D printer OctoEverywhere supports.

## Calling the APIs

The plugin APIs can be called using a [Shared Connection](https://octoeverywhere.com/sharedconnections?source=docs_plugin_api) or [App Connection](../app-connections/index.md) URL, using:

`https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/...`

Where `<unique_id>` is your unique shared connection or app connection subdomain.

## HTTP Error Codes

Plugin APIs return common [OctoEverywhere Error Codes](../error-codes.md) for OctoEverywhere service-side issues and standard HTTP error codes for plugin API issues.

## Plugin APIs

### [Printer Status](printer-status.md)

A printer-agnostic JSON object with current printer status, print progress, percentage complete, estimated time remaining, and more.

### [Printer Control](printer-control.md)

Printer-agnostic control for pausing or canceling prints, toggling lights, homing axes, extruding filament, and more.

### [Webcam APIs](webcam-api.md)

A set of printer-agnostic webcam APIs, including camera lists, multi-camera snapshots, and multi-camera streams.

### [MQTT over WebSocket Proxy](mqtt-websocket-proxy.md)

**Works With:** Bambu Lab & Elegoo Centauri Carbon 2

Bambu Lab and Elegoo Centauri Carbon 2 printers use MQTT for low-level status, control, and event streams. OctoEverywhere provides a secure, internet-facing, standards-compliant MQTT-over-WebSocket proxy so apps can use normal MQTT client libraries to reach the printer from anywhere.

The original JSON-based WebSocket transport is still available for existing integrations, but new clients should use the standards-compliant proxy.

[Legacy JSON Transport Reference](mqtt-websocket-proxy-json-legacy.md)
