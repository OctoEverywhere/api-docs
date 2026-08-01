---
title: 3D Printer Webcam API
description: Get live webcam snapshots and streams from OctoPrint, Moonraker, Bambu Lab, PrusaLink, Elegoo, and more through one OctoEverywhere API.
og_title: Stream 3D Printer Webcams from Anywhere
og_description: Add remote webcam snapshots and MJPEG streams to your app with one OctoEverywhere API that works across popular 3D printer platforms.
authors:
    - Quinn Damerell
date: 2026-07-31
---

# Plugin Webcam API

OctoEverywhere's Plugin Webcam API is a platform-agnostic 3D printer API that provides webcam snapshots and streams from OctoPrint, Moonraker, Bambu Lab, PrusaLink, and other supported platforms through one common API surface.

!!! tip
    These APIs work with every 3D printer OctoEverywhere supports, including OctoPrint, Moonraker, Klipper, Bambu Lab, Prusa, Elegoo, Creality, and more.

[Get Started With Plugin APIs](overview.md){ .md-button .md-button--primary }

## Common Error Codes

All Plugin APIs share a set of [common error codes](./plugin-api-errors.md) that can be returned for issues like the OctoEverywhere plugin is offline, auth issues, etc.

## List Webcams

Returns a list of all webcams the user has set up in the 3D printer host software. The array index is used as the `webcam index` for the other API calls.

```{.http .apirequest title="HTTP Request"}
GET https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/webcam/list
```

```{.json .apiresponse title="Example Response"}
{
    "Status": 200,
    "Result": {
        "DefaultIndex": 0,
        "Webcams": [
            {
                "Name": "Webcam Name",
                "FlipH": false,
                "FlipV": false,
                "Rotation": 0,
                "Enabled": true,
                "SnapshotUrl": "/webcam/?action=snapshot",
                "StreamUrl": "/webcam/?action=stream"
            }
        ]
    }
}
```

| Name | Type | Description |
| ---- | :--: | ----------- |
| `DefaultIndex` | int | Index selected when a snapshot or stream request omits `index`. |
| `Webcams` | list | Webcams in the same index order used by the snapshot and stream commands. |
| `Name` | string | User-facing webcam name. |
| `FlipH` | bool | Whether clients should horizontally flip the image. |
| `FlipV` | bool | Whether clients should vertically flip the image. |
| `Rotation` | int | Clockwise display rotation: `0`, `90`, `180`, or `270`. |
| `Enabled` | bool | Whether the webcam is enabled in the current plugin configuration. |
| `SnapshotUrl` | string \| null | Platform-local snapshot URL, when available. Use the OctoEverywhere snapshot command for remote access. |
| `StreamUrl` | string \| null | Platform-local stream URL, when available. Use the OctoEverywhere stream command for remote access. |

The webcam summary embedded in the [Printer Status API](printer-status.md) deliberately omits `SnapshotUrl` and `StreamUrl` to keep the status response small. The standalone `webcam/list` command includes them.


## Get Webcam Snapshot

Returns a JPEG snapshot from the webcam.


```{.http .apirequest title="HTTP Request"}
GET https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/webcam/snapshot?index=0
```

| Name       |  Type  | Default              | Description                                         |
| ---------- | :----: | -------------------- | --------------------------------------------------- |
| `index` | int | `DefaultIndex` | Webcam index to capture. When omitted, the value returned by `webcam/list` is used. |



## Get Webcam Stream

Returns an MJPEG stream from the webcam. The stream is a series of JPEG images returned by the HTTP request.

```{.http .apirequest title="HTTP Request"}
GET https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/webcam/stream?index=0
```

| Name       |  Type  | Default              | Description                                         |
| ---------- | :----: | -------------------- | --------------------------------------------------- |
| `index` | int | `DefaultIndex` | Webcam index to stream. When omitted, the value returned by `webcam/list` is used. |
