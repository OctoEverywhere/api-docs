# 🌍 3D Printer API Remote Access

OctoEverywhere provides secure remote access to any 3D printer's HTTP REST and WebSocket APIs. This gives you full access to your 3D printer from any service or app on the public internet.

## Works With

Our API remote access works with any 3D printer supported by OctoEverywhere, including, but not limited to:

- OctoPrint
- Klipper / Moonraker
- PrusaLink
- Bambu Lab
- Elegoo CC1 and CC2
- And more

## Remote Access Vs Plugin APIs

OctoEverywhere supports full remote access directly to the printers HTTP REST, WebSocket, and MQTT servers, but it also has platform agnostic plugin APIs that make targeting multiple printer platforms easy.

#### API, WebSocket, MQTT Remote Access

This gives you direct remote access to the full HTTP REST, WebSocket, or MQTT interface. Use this system if you want to talk directly to OctoPrint, Moonraker, etc directly. You can set any headers needed, send raw MQTT commands, use WebSocket and MQTT based subscriptions and status update pushes, etc.

#### Plugin APIs

Use the [OctoEverywhere Plugin APIs](../plugin-api/) for platform agnostic and easy access to the 3D printer. The API surface is common for all 3D printer types, and allows access to the printer status, control, commands, webcam streaming, webcam snapshots, file listing, upload, download, and more!


!!! tip
    For some 3D printer platforms, like Bambu Lab, the plugin APIs are the only way to access things like the MQTT server and file upload and download.


## Get Started

Full docs are coming soon. Here's the quick version:

- [Add the OctoEverywhere plugin to your 3D printer](https://octoeverywhere.com/getstarted?source=docs)
- Create a [Shared Connection](https://octoeverywhere.com/sharedconnections?source=docs).
- Use any REST API or WebSocket connection that would normally work directly with the 3D printer software. Replace the local IP address with your unique Shared Connection subdomain.

Our relay service supports all HTTP request types (GET, POST, etc.) and WebSockets.

## MQTT Proxy Relay

Our new MQTT relay enables access to your 3D printer's full MQTT server via the public internet. It uses a secure, standards-compliant MQTT-over-WebSocket transport, which is supported by most common MQTT clients.

[Learn More](../plugin-api/mqtt-websocket-proxy.md){ .md-button .md-button--primary }
