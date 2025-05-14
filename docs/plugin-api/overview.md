# Plugin API Overview

## What is it?

The OctoEverywhere plugin provides a set of useful APIs that can be accessed from any OctoEverywhere remote access connection. The plugin APIs can be used from:

- Shared Connections
- App Connections

## Root Path

This is the common root path shared by all plugin APIs.

`https://<host>/octoeverywhere-command-api/...`

## HTTP Error Codes

[OctoEverywhere Error Codes](../error-codes.md) can be returned from any API to indicate errors.

## APIs

### [Webcam APIs](webcam-api.md)

**Works With:** All OctoEverywhere Plugins

The plugin webcam APIs create a printer host-agnostic set of webcam APIs, including listing webcams, getting multi-cam snapshots, and multi-cam streams - making accessing the webcam on different types 3D printers easy.


### [MQTT Websocket Proxy](mqtt-websocket-proxy.md)

**Works With:** Bambu Lab OctoEverywhere Plugins

Bambu Lab 3D printers use MQTT as their primary control and monitoring protocol. Since working with MQTT is tricky on the web, the OctoEverywhere plugin has a full MQTT proxy implementation using WebSockets.