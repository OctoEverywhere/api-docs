---
title: Webhook JSON Payload Format
description: Understand the JSON payload OctoEverywhere sends to webhook endpoints, including printer IDs, print details, progress, snapshots, and event data.
og_title: Parse OctoEverywhere Webhook Payloads
og_description: See the fields OctoEverywhere sends with each notification so your app can handle printer events, progress, snapshots, file names, and print timing.
---

# Webhook JSON Payload Format

## Overview

This is the JSON payload sent by the OctoEverywhere service as the body payload for every Webhook callback POST. 

These payloads are common to both [Notification webhooks](./overview.md) and [App Connection Webhooks](../app-connections/apis/notification-webhook.md).

```{.json .apiresponse title="Example Webhook POST JSON Payload"}
{
    "PrinterId": "Id",
    "SecretKey": "Key",
    "PrintId": "Id",
    "EventType": 1,
    "PrinterName": "Ender3",
    "SnapshotUrl": "https://...",
    "QuickViewUrl": "https://...",
    "FileName": "MyCoolPrint",
    "DurationSec": 50,
    "Progress": 21,
    "TimeRemainingSec": 605,
    "ZOffsetMM": 300,
    "CurrentLayer": 50,
    "TotalLayers": 100,
    "Error": "Oh no, something bad happened!",
    "PlatformErrorCode": "33"
    ...
}
```


!!! note
    These JSON keys will never be removed or changed, but overtime new properties will be added!

## Payload Details

The HTTP POST request includes a JSON body with these properties:

- **PrinterId**
    - (string) A stable unique ID for your printer. This ID will not change as long as your printer remains attached to your account.
- **SecretKey**
    - (string) If you set a secret key, it will be included here.
- **PrintId**
    - (string) A unique string for each print. This string is created when a print starts and remains until the print is complete or stopped, making it useful for tracking which notifications are associated with which print jobs.
- **EventType**
    - (int, enum) This enum maps to the notification type. [The enum is defined here.](event-types.md)
- **PrinterName**
    - (string) The name you assigned your printer. This name will change if the printer is renamed on OctoEverywhere.
- **SnapshotUrl**
	- (string, optional) If a snapshot can be captured, this will be a URL where the image can be viewed or downloaded. This image URL will remain valid for about 7 days.
- **QuickViewUrl**
    - (string) A URL to OctoEverywhere's Quick View, which provides a secure, internet-based way to quickly view the full printer state, pause prints, and cancel prints.
- **FileName**
	- (string, optional) The file name in OctoPrint for the current file being printed.
- **DurationSec**
	- (int, optional) The duration of the print since it started, in seconds.
- **Progress**
	- (int, optional) The current print progress as a percentage, where 0 is 0% and 100 is 100%.
- **TimeRemainingSec**
	- (int, optional) The estimated remaining print time reported by OctoPrint, in seconds.
- **ZOffsetMM**
	- (int, optional) The current Z-axis offset in millimeters.
- **CurrentLayer**
	- (int, optional) The current layer count being printed.
- **TotalLayers**
	- (int, optional) The total number of layers in the model being printed.
- **Error**
	- (string, optional) If set, this contains an platform specific error message.
- **PlatformErrorCode**
	- (string, optional) If set, this contains an platform specific error code. The code format is taken directly format is taken directly from the 3D printer platform.

