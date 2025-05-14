# Plugin Webcam API

This page is still being built, but here's a quick overview.

Remember all plugin APIs share a [common root path](./overview.md) and [custom error codes.](./overview.md)

## List Webcams

Returns a list of all webcams the user has setup in the 3D printer host software. Note the array index is used as the `webcam index` for the other API calls.

```{.http .apirequest title="HTTP Request"}
GET https://<host>/octoeverywhere-command-api/webcam/list
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

Returns a .jpg snapshot from the webcam.


```{.http .apirequest title="HTTP Request"}
GET https://<host>/octoeverywhere-command-api/webcam/snapshot?index=0
```

/// api-parameters
    open: True

| Name       |  Type  | Default              | Description                                         |
| ---------- | :----: | -------------------- | --------------------------------------------------- |
| `index` | int | 0          | The index of the webcam to get a snapshot from.                            |

///



## Get Webcam Stream

Returns a mjpeg stream of the webcam. The mjpeg stream is a series of jpeg images streamed as the result of the HTTP request.

```{.http .apirequest title="HTTP Request"}
GET https://<host>/octoeverywhere-command-api/webcam/stream?index=0
```

/// api-parameters
    open: True

| Name       |  Type  | Default              | Description                                         |
| ---------- | :----: | -------------------- | --------------------------------------------------- |
| `index` | int | 0          | The webcam index to get a stream from.                            |

///