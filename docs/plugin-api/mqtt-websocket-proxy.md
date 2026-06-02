---
title: MQTT over WebSocket Remote Access
description: Secure standards-compliant MQTT-over-WebSocket proxy for full Bambu Lab and Elegoo CC2 3D printer MQTT access from anywhere.
authors:
    - Quinn Damerell
date: 2026-06-01
---

# MQTT over WebSocket Remote Access

OctoEverywhere's MQTT WebSocket Proxy exposes your 3D printer's MQTT server through a secure, internet-facing `wss://` endpoint. Apps can subscribe, publish, and receive printer MQTT messages from anywhere without opening a port, running a VPN, or exposing the printer's local MQTT broker to the public internet.

This enables full MQTT access to Bambu Lab printers and Elegoo Centauri Carbon 2 (CC2), using any MQTT client, service, AI, or app.

## MQTT Proxy Features

- **Secure internet-facing MQTT:** Give apps and services public-internet access to printer MQTT without exposing the printer, the local network, or the printer's MQTT broker directly.
- **Standards-compliant transport:** Use MQTT over WebSocket with common MQTT client libraries instead of a custom message protocol, so it works with anything.
- **Full printer access:** Subscribe and publish to the same printer-specific MQTT topics your client would use on the local network.
- **Printer credentials stay local:** OctoEverywhere handles the printer-side MQTT authentication through the user's installed plugin connection.
- **Shared MQTT connection:** All MQTT connections share one common MQTT client connection to the 3D printer, which is great for printers with limited concurrent-client support.

## Supported Printers

- All Bambu Lab 3D printers
- Elegoo Centauri Carbon 2

## Connection Endpoint

The MQTT WebSocket Proxy can be used from a [Shared Connection](https://octoeverywhere.com/sharedconnections?source=docs_plugin_api_mqtt) or [App Connection](../app-connections/index.md) URL.

```{.http .apirequest}
wss://<unique-id>.octoeverywhere.com/octoeverywhere-command-api/proxy/mqtt
```

## Authentication

No MQTT authentication credentials are required, but authentication to OctoEverywhere is.

The local MQTT authentication is handled by the OctoEverywhere plugin, and the OctoEverywhere service WebSocket connection is handled using the existing [Shared Connection](https://octoeverywhere.com/sharedconnections?source=docs_plugin_api_mqtt_auth) or [App Connection](../app-connections/index.md) authentication.

Shared Connection or App Connection authentication is usually sent through HTTP headers, which some MQTT clients may not support. In that case, send the `BearerToken` or HTTP `username` and `password` digest as a query parameter named `oe_api_key`.

```{.http .apirequest}
wss://.../octoeverywhere-command-api/proxy/mqtt?oe_api_key=<key>
```

The `oe_api_key` is formatted depending on the auth type:

- `HTTP Basic Access` - The `oe_api_key` value must be the base64-encoded HTTP `username` and `password` result.
- `HTTP Bearer Token` - The `oe_api_key` value must be only the Bearer token returned from the app portal, with nothing else appended to it.

## Error Codes

When your client opens the WebSocket, the proxy starts and connects the MQTT session to the printer automatically. If the printer's MQTT connection fails, the WebSocket closes with one of the OctoEverywhere [common error codes](./../error-codes.md).

## MQTT Standards

The OctoEverywhere MQTT WebSocket proxy fully supports the MQTT 3.1 over WebSocket standard. Since supported 3D printers use MQTT 3.1 today, this keeps the proxy broadly compatible.

Almost all MQTT clients and code libraries support MQTT 3.1 over a WebSocket transport.

| Setting | Value |
| ------- | ----- |
| WebSocket scheme | `wss` |
| Host | Your App Connection or Shared Connection host |
| Path | `/octoeverywhere-command-api/proxy/mqtt` |
| MQTT Credentials | None |
| Headers/HTTP GET Auth | Same as the App or Shared Connection auth |
