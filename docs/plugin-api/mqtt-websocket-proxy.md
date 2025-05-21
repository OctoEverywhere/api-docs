# MQTT Remote Access Proxy

OctoEverywhere's [Bambu Connect](https://octoctoeverywhere.com/bambu?source=docs) is our plugin for Bambu Lab 3D printers.

Bambu Lab 3D printers work differently from OctoPrint, Klipper, and Prusa Connect 3D printers. They rely on a protocol called MQTT to command and control the printer.

MQTT is a lightweight protocol on top of TCP, which makes remote access tricky. However, we developed a WebSocket proxy for MQTT to allow Bambu Lab app developers to use OctoEverywhere's [App Connections](../app-connections/index.md) remote access.

Our MQTT proxy can be almost like an MQTT tunnel through a web socket. We designed the messages to be very similar to the MQTT messages, so only minor changes have to be made to swap our WebSocket proxy for an actual MQTT implementation.


## Connection & Authentication

### Websocket Endpoint

The OctoEverywhere MQTT Websocket Proxy can be used from any [App Connection](../app-connections/index.md)  URL or [Shared Connection](https://octoeverywhere.com/sharedconnection) URL.

```{.http .apirequest title="Secure WebSocket Endpoint"}
wss://app-<id>.octoeverywhere.com/octoeverywhere-command-api/proxy/mqtt
```

For the WebSocket authentication to OctoEverywhere's servers, use the same auth headers that you would use for other [App Connection](../app-connections/index.md) HTTP or WebSocket requests.

When your client makes a WebSocket connection to the OctoEverywhere endpoint, the proxy MQTT connection will automatically be started, connected, and ready to use. If the printer's MQTT connection fails, the WebSocket will close with one of the OctoEverywhere [common error codes.](./../error-codes.md)

### Bambu Lab MQTT Authentication

The Bambu Lab 3D printer's MQTT authentication is handled for you! The OctoEverywhere plugin will automatically use the same Bambu Lab serial number and access code that the user set for the OctoEverywhere plugin connection.

## WebSocket MQTT Protocol

Our WebSocket proxy protocol attempts to mirror MQTTs protocol as closely as possible. Thus, concepts like subscribing and publishing work the same way.

### Message Flow

Here's an example of what the MQTT websocket proxy messages might look like when connecting to a Bambu Lab 3D printer.

```
Your App             (Websocket MQTT Proxy)    Bambu Lab 3D Printer MQTT Server

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

The `subscribe` message can have an ID your client randomly generates which will then be paired with the `subscribe_ack`, allowing you to match the `subscribe_ack` and `subscribe_ack` messages. The `subscribe_ack` will include the MQTT MID (message id). When the MQTT server sends back the `on_subscribe` event, it will also include a MQTT MID, thus you are able to link each `subscribe` request message with the resulting `on_subscribe` event.

After your app is subscribed to a topic, the server will start sending `on_message` messages whenever a topic update occurs. For Bambu Lab 3D printers, these messages include printer-state changes.

If you want to send a command, your app sends the `publish` request message, which the MQTT Websocket Proxy will give you as the end result of using `publish_ack`. The `publish_ack` can be used to verify the publish message was forwarded successfully.


### Message Concepts

As you can see in the example above, there are two message concepts for our WebSocket proxy.

1. Remote Function Calls Style Messages
    - A "remote function call" (RPC) type message where you send a request and get an ack response back:
        - `subscribe` -> `subscribe_ack`
        - `unsubscribe` -> `unsubscribe_ack`
        - `publish` -> `publish_ack`

2. Event Style Messages
    - These are messages from the MQTT protocol, which the server sends as responses to function calls and/or streaming messages.
        - `on_message`
        - `on_subscribe`
        - `on_unsubscribe`

### RPC Style Messages

#### Subscribe

Here's an example of a  `subscribe` request message and the `subscribe_ack` response.

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
| `Id` | int |        Optional - A client created random int that will be repeated back to match the `subscribe_ack` response. |
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
| `AckResult` | int | The MQTT error code of the action. `0` indicates success, all other values are errors.                     |
| `MqttMessageId` | int | The MQTT MID (Message Id) for the subscribe request. |
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
| `Type` | string  |  Must be the action: `subscribe`.                            |
| `Topic` | string | The MQTT topic to subscribe to.                     |
| `Id` | int |        Optional - A client created random int that will be repeated back to match the `subscribe_ack` response. |
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
| `AckResult` | int | The MQTT error code of the action. `0` indicates success, all other values are errors.                     |
| `MqttMessageId` | int | The MQTT MID (Message Id) for the subscribe request. |
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
| `Payload` | string | Any binary data base64 encoded as a string. (usually a json string)           |
| `Id` | int |        Optional - A client created random int that will be repeated back to match the `subscribe_ack` response. |
| `NoAck` | bool |    Optional - If set to true, no `unsubscribe_ack` response will be sent.           |

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
| `AckResult` | int | The MQTT error code of the action. `0` indicates success, all other values are errors.                     |
| `MqttMessageId` | int | The MQTT MID (Message Id) for the subscribe request. |
| `Id` | int |    Optional - If a random `Id` was sent in the `publish` request, the same `Id` will be returned here for matching.        |

///


### Event Messages

#### OnMessage

Here's an example `on_message` event message sent from the MQTT server. These messages are sent as updates once a topic has been subscribed to.

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
| `Payload` | string | Any binary data base64 encoded as a string. (usually a json string)           |
| `MqttMessageId` | int | The MQTT MID (Message Id) from the MQTT server. |
///


#### OnSubscribe

Here's an example `on_subscribe` event message sent from the MQTT server. This is sent by the MQTT server in response to a `subscribe` request.

```{.json .apirequest title="WebSocket Proxy on_message"}
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
| `Type` | string  |  Must be the action: `on_message`.                            |
| `ReasonCodeList` | list [int] | A list of MQTT version 5.0 reason codes. Code `0` is success, all others are failures.           |
| `MqttMessageId` | int | The MQTT MID (Message Id) from the MQTT server. |
| `Id` | int |    Optional - If a random `Id` was sent in the `subscribe` request, the same `Id` will be returned here for matching.        |
///

#### OnUnsubscribe

Here's an example `on_unsubscribe` event message sent from the MQTT server. This is sent by the MQTT server in response to a `unsubscribe` request.

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
| `ReasonCodeList` | list [int] | A list of MQTT version 5.0 reason codes. Code `0` is success, all others are failures.           |
| `MqttMessageId` | int | The MQTT MID (Message Id) from the MQTT server. |
| `Id` | int |    Optional - If a random `Id` was sent in the `unsubscribe` request, the same `Id` will be returned here for matching.        |
///




