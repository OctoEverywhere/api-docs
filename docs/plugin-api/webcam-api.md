---
title: 3D Printing Webcam API
description: OctoEverywhere's webcam API lets you capture snapshots or stream your webcam from anywhere!
authors:
    - Quinn Damerell
date: 2025-05-20
---

# Plugin Webcam API

This page is still being built. Here's a quick overview.

Remember, all plugin APIs share a [common root path](./index.md) and [custom error codes](./../error-codes.md).

!!! tip
    This generic webcam API works with every 3D printer OctoEverywhere supports, including OctoPrint, Moonraker, Klipper, Bambu Lab, Prusa, Elegoo, Creality, and more!

## List Webcams

Returns a list of all webcams the user has set up in the 3D printer host software. The array index is used as the `webcam index` for the other API calls.

```{.http .apirequest title="HTTP Request"}
GET https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/webcam/list
```

```{.json .apiresponse title="Example Response"}
{
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
