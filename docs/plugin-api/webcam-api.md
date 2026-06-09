---
title: 3D Printer Webcam API
description: OctoEverywhere's webcam API lets you get webcam snapshots and streams from OctoPrint, Moonraker, Bambu Lab, PrusaLink and more from one common API!
authors:
    - Quinn Damerell
date: 2025-05-20
---

# Plugin Webcam API

OctoEverywhere's Plugin Webcam API is a platform-agnostic 3D printer API that allows access webcam snapshots and streams from OctoPrint, Moonraker, Bambu Lab, and PrusaLink with a single common API surface.

!!! tip
    These APIs work with every 3D printer OctoEverywhere supports, including OctoPrint, Moonraker, Klipper, Bambu Lab, Prusa, Elegoo, Creality, and more.

[Get Started With Plugin APIs](index.md){ .md-button .md-button--primary }

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
    "Result:": {
        "DefaultIndex": 0,
        "Webcams": [
            {
                "Name": "Webcam Name",
                "FlipH": false,
                "FlipV": false,
                "Rotation": 0,
                "Enabled": true
            }
            ...
        ]
    }
}
```


## Get Webcam Snapshot

Returns a JPEG snapshot from the webcam.


```{.http .apirequest title="HTTP Request"}
GET https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/webcam/snapshot?index=0
```

| Name       |  Type  | Default              | Description                                         |
| ---------- | :----: | -------------------- | --------------------------------------------------- |
| `index` | int | 0          | The webcam index to capture a snapshot from.                            |



## Get Webcam Stream

Returns an MJPEG stream from the webcam. The stream is a series of JPEG images returned by the HTTP request.

```{.http .apirequest title="HTTP Request"}
GET https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/webcam/stream?index=0
```

| Name       |  Type  | Default              | Description                                         |
| ---------- | :----: | -------------------- | --------------------------------------------------- |
| `index` | int | 0          | The webcam index to stream from.                            |
