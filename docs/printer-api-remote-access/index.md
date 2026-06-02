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

## Get Started

Full docs are coming soon. Here's the quick version:

- [Add the OctoEverywhere plugin to your 3D printer](https://octoeverywhere.com/getstarted?source=docs)
- Create a [Shared Connection](https://octoeverywhere.com/sharedconnections?source=docs).
- Use any REST API or WebSocket connection that would normally work directly with the 3D printer software. Replace the local IP address with your unique Shared Connection subdomain.

Our relay service supports all HTTP request types (GET, POST, etc.) and WebSockets.

## MQTT Proxy Relay

Our new MQTT relay enables access to your 3D printer's full MQTT server via the public internet. It uses a secure, standards-compliant MQTT-over-WebSocket transport, which is supported by most common MQTT clients.

[Learn More](../plugin-api/mqtt-websocket-proxy.md){ .md-button .md-button--primary }
