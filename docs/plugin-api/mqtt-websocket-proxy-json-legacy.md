---
title: Legacy JSON MQTT WebSocket Transport
description: Legacy OctoEverywhere JSON MQTT WebSocket transport for backward-compatible Bambu Lab and Elegoo CC2 access.
authors:
    - Quinn Damerell
date: 2025-05-20
---

# Legacy JSON MQTT WebSocket Transport

!!! warning "Legacy transport"
    This JSON-based transport is kept for backward compatibility with existing clients. New clients should use the [standards-compliant MQTT over WebSocket transport](mqtt-websocket-proxy.md), which uses normal MQTT frames and works with common MQTT client libraries.

This page documents OctoEverywhere's original JSON message protocol for the MQTT WebSocket Proxy. It runs on the same secure endpoint and uses the same OctoEverywhere authentication as the recommended MQTT-over-WebSocket transport, but the messages are OctoEverywhere-specific JSON objects instead of standards-compliant MQTT frames.

The proxy automatically detects the transport from the first client message:

| First client message | Proxy mode |
| -------------------- | ---------- |
| MQTT frame, such as an MQTT `CONNECT` packet | Standards-compliant MQTT over WebSocket |
| JSON message, such as `{ "Type": "subscribe" }` | Legacy OctoEverywhere JSON transport |

The legacy JSON protocol can provide full MQTT access to supported Bambu Lab and Elegoo Centauri Carbon 2 (CC2) printers, but it should only be used to maintain existing integrations.

## Connection and Authentication

### WebSocket Endpoint

The OctoEverywhere MQTT WebSocket Proxy can be used from any [App Connection](../app-connections/index.md) URL or [Shared Connection](https://octoeverywhere.com/sharedconnections?source=docs_plugin_api_legacy_mqtt) URL.

```{.http .apirequest title="Secure WebSocket Endpoint"}
wss://<app-or-shared-connection-host>/octoeverywhere-command-api/proxy/mqtt
```

For WebSocket authentication to OctoEverywhere's servers, use the same auth headers that you would use for other [App Connection](../app-connections/index.md) HTTP or WebSocket requests.

When your client makes a WebSocket connection to the OctoEverywhere endpoint, the proxy starts and connects the MQTT session automatically. If the printer's MQTT connection fails, the WebSocket will close with one of the OctoEverywhere [common error codes](./../error-codes.md).

### Printer MQTT Authentication

The printer-side MQTT authentication is handled for you. OctoEverywhere uses the printer credentials already configured in the user's OctoEverywhere plugin connection, such as the Bambu Lab serial number and access code for Bambu Lab printers.

## Legacy JSON WebSocket Protocol

The legacy JSON protocol mirrors MQTT concepts like subscribing and publishing, but each action is wrapped in an OctoEverywhere-specific JSON message.

### Message Flow

Here's an example of what the legacy JSON proxy messages might look like when connecting to a printer MQTT server.

```
Your App             (WebSocket MQTT Proxy)    Printer MQTT Server

subscribe->          subscribe->               subscribe->
<-subscribe_ack      <-subscribe_ack
...
<-on_subscribe       <-on_subscribe            <-on_subscribe
...
<-on_message         <-on_message              <-on_message
<-on_message         <-on_message              <-on_message
...
publish->            publish->                 publish->
<-publish_ack        <-publish_ack
...
<-on_message         <-on_message              <-on_message
...
```

The `subscribe` message can include a client-generated `Id`. The matching `subscribe_ack` echoes that value, making it easy to pair requests and acknowledgments. The `subscribe_ack` also includes the MQTT MID (message ID). When the MQTT server sends back the `on_subscribe` event, it includes an MQTT MID as well, allowing you to link each `subscribe` request with the resulting `on_subscribe` event.

After your app is subscribed to a topic, the server will start sending `on_message` messages whenever a topic update occurs. For supported printers, these messages include printer-state changes.

If you want to send a command, your app sends the `publish` request message. The MQTT WebSocket Proxy returns `publish_ack` after the publish message has been forwarded successfully.


### Message Concepts

As shown above, the legacy JSON proxy uses two message concepts.

1. Remote Function Call Style Messages
    - A "remote function call" (RPC) style message where you send a request and receive an acknowledgment response:
        - `subscribe` -> `subscribe_ack`
        - `unsubscribe` -> `unsubscribe_ack`
        - `publish` -> `publish_ack`

2. Event Style Messages
    - These are messages from the MQTT protocol that the server sends as responses to function calls and/or as streaming messages.
        - `on_message`
        - `on_subscribe`
        - `on_unsubscribe`

### RPC Style Messages

#### Subscribe

Here's an example of a `subscribe` request message and the `subscribe_ack` response.

```{.json .apirequest title="WebSocket Proxy subscribe"}
{
    "Type": "subscribe",
    "Topic": "device/serial_number/report",
    "Id": 1337,
    "NoAck": false
}

```
/// api-parameters
    open: True

| Name       |  Type  |  Description                                         |
| ---------- | :----: | --------------------------------------------------- |
| `Type` | string  |  Must be the action: `subscribe`.                            |
| `Topic` | string | The MQTT topic to subscribe to.                     |
| `Id` | int |        Optional - A client-created random integer that will be repeated back to match the `subscribe_ack` response. |
| `NoAck` | bool |    Optional - If set to true, no `subscribe_ack` response will be sent.           |

///


```{.json .apirequest title="WebSocket Proxy subscribe_ack"}
{
    "Type": "subscribe_ack",
    "AckResult": 0,
    "MqttMessageId": 222,
    "Id": 1337,
}
```
/// api-parameters
    open: False

| Name       |  Type  |  Description                                         |
| ---------- | :----: | --------------------------------------------------- |
| `Type` | string  |  Must be the request ack: `subscribe_ack`.                            |
| `AckResult` | int | The MQTT result code for the action. `0` indicates success; all other values are errors.                     |
| `MqttMessageId` | int | The MQTT MID (message ID) for the subscribe request. |
| `Id` | int |    Optional - If a random `Id` was sent in the `subscribe` request, the same `Id` will be returned here for matching.        |

///



#### Unsubscribe

Here's an example `unsubscribe` request message and the `unsubscribe_ack` response.

```{.json .apirequest title="WebSocket Proxy unsubscribe"}
{
    "Type": "unsubscribe",
    "Topic": "device/serial_number/report",
    "Id": 2000,
    "NoAck": false
}

```
/// api-parameters
    open: False

| Name       |  Type  |  Description                                         |
| ---------- | :----: | --------------------------------------------------- |
| `Type` | string  |  Must be the action: `unsubscribe`.                            |
| `Topic` | string | The MQTT topic to unsubscribe from.                     |
| `Id` | int |        Optional - A client-created random integer that will be repeated back to match the `unsubscribe_ack` response. |
| `NoAck` | bool |    Optional - If set to true, no `unsubscribe_ack` response will be sent.           |

///


```{.json .apirequest title="WebSocket Proxy unsubscribe_ack"}
{
    "Type": "unsubscribe_ack",
    "AckResult": 0,
    "MqttMessageId": 223,
    "Id": 2000,
}
```
/// api-parameters
    open: False

| Name       |  Type  |  Description                                         |
| ---------- | :----: | --------------------------------------------------- |
| `Type` | string  |  Must be the request ack: `unsubscribe_ack`.                            |
| `AckResult` | int | The MQTT result code for the action. `0` indicates success; all other values are errors.                     |
| `MqttMessageId` | int | The MQTT MID (message ID) for the unsubscribe request. |
| `Id` | int |    Optional - If a random `Id` was sent in the `unsubscribe` request, the same `Id` will be returned here for matching.        |

///

#### Publish

Here's an example `publish` request message and the `publish_ack` response.

```{.json .apirequest title="WebSocket Proxy publish"}
{
    "Type": "publish",
    "Topic": "device/serial_number/report",
    "Payload": "base64 encoded data",
    "Id": 5000,
    "NoAck": false
}

```
/// api-parameters
    open: False

| Name       |  Type  |  Description                                         |
| ---------- | :----: | --------------------------------------------------- |
| `Type` | string  |  Must be the action: `publish`.                            |
| `Topic` | string | The MQTT topic to publish to.                     |
| `Payload` | string | Base64-encoded binary data as a string, usually a JSON string.           |
| `Id` | int |        Optional - A client-created random integer that will be repeated back to match the `publish_ack` response. |
| `NoAck` | bool |    Optional - If set to true, no `publish_ack` response will be sent.           |

///


```{.json .apirequest title="WebSocket Proxy publish_ack"}
{
    "Type": "publish_ack",
    "AckResult": 0,
    "MqttMessageId": 224,
    "Id": 5000,
}
```
/// api-parameters
    open: False

| Name       |  Type  |  Description                                         |
| ---------- | :----: | --------------------------------------------------- |
| `Type` | string  |  Must be the request ack: `publish_ack`.                            |
| `AckResult` | int | The MQTT result code for the action. `0` indicates success; all other values are errors.                     |
| `MqttMessageId` | int | The MQTT MID (message ID) for the publish request. |
| `Id` | int |    Optional - If a random `Id` was sent in the `publish` request, the same `Id` will be returned here for matching.        |

///


### Event Messages

#### OnMessage

Here's an example `on_message` event message sent from the MQTT server. These messages are sent as updates after a topic has been subscribed to.

```{.json .apirequest title="WebSocket Proxy on_message"}
{
    "Type": "on_message",
    "Payload": "base64 encoded data",
    "MqttMessageId": 255,
}

```
/// api-parameters
    open: True
| Name       |  Type  |  Description                                         |
| ---------- | :----: | --------------------------------------------------- |
| `Type` | string  |  Must be the action: `on_message`.                            |
| `Payload` | string | Base64-encoded binary data as a string, usually a JSON string.           |
| `MqttMessageId` | int | The MQTT MID (message ID) from the MQTT server. |
///


#### OnSubscribe

Here's an example `on_subscribe` event message sent from the MQTT server in response to a `subscribe` request.

```{.json .apirequest title="WebSocket Proxy on_subscribe"}
{
    "Type": "on_subscribe",
    "ReasonCodeList": [0, ...],
    "MqttMessageId": 255,
    "Id": 5000,
}

```
/// api-parameters
    open: False
| Name       |  Type  |  Description                                         |
| ---------- | :----: | --------------------------------------------------- |
| `Type` | string  |  Must be the action: `on_subscribe`.                            |
| `ReasonCodeList` | list \[int\] | A list of MQTT version 5.0 reason codes. Code `0` means success; all others are failures.           |
| `MqttMessageId` | int | The MQTT MID (message ID) from the MQTT server. |
| `Id` | int |    Optional - If a random `Id` was sent in the `subscribe` request, the same `Id` will be returned here for matching.        |
///

#### OnUnsubscribe

Here's an example `on_unsubscribe` event message sent from the MQTT server in response to an `unsubscribe` request.

```{.json .apirequest title="WebSocket Proxy on_unsubscribe"}
{
    "Type": "on_unsubscribe",
    "ReasonCodeList": [0, ...],
    "MqttMessageId": 255,
    "Id": 5000,
}

```
/// api-parameters
    open: False
| Name       |  Type  |  Description                                         |
| ---------- | :----: | --------------------------------------------------- |
| `Type` | string  |  Must be the action: `on_unsubscribe`.                            |
| `ReasonCodeList` | list \[int\] | A list of MQTT version 5.0 reason codes. Code `0` means success; all others are failures.           |
| `MqttMessageId` | int | The MQTT MID (message ID) from the MQTT server. |
| `Id` | int |    Optional - If a random `Id` was sent in the `unsubscribe` request, the same `Id` will be returned here for matching.        |
///




