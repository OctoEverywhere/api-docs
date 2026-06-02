---
title: Process API
description: Process a print snapshot through OctoEverywhere AI failure detection.
authors:
    - Quinn Damerell
date: 2025-05-20
---

# Process API

The Process API is where the AI does the heavy lifting.

This API must be called with a context created by the [Create Context API](create-context.md). Each request sends a recent JPEG image of the print. OctoEverywhere's ML image models process the image, then the temporal combination model returns output based on the current image, previous context, and other signals.

!!! tip
    Process API usage is counted as part of [AI failure detection pricing](index.md#pricing). Every developer gets 5,000 free calls per month, which is about 55 hours of printing.

## HTTP Request

Use the `ProcessRequestUrl` returned by the [Create Context API](create-context.md). If that URL fails, switch to the returned `FallbackProcessRequestUrl` for the rest of the context lifetime.

```{.http .apirequest title="HTTP Request"}
POST https://<your-process-host>.octoeverywhere.com/api/gadget/v1/process/{ID}
```

## Headers


| Name        | Type   | Required | Description |
| ----------- | :----: | :------: | ----------- |
| `X-API-Key` | string | Yes      | Your OctoEverywhere developer API key. |


## Path Parameters

| Name | Type   | Required | Description |
| ---- | :----: | :------: | ----------- |
| `ID` | string | Yes      | The context ID returned by the [Create Context API](create-context.md). |


## Request Body

Send the image as a `multipart/form-data` POST body. Include exactly one attached file; the uploaded file name does not matter.

This request type is supported by common HTTP libraries in modern languages. The [Python SDK](https://github.com/OctoEverywhere/Gadget-Python-Sdk/blob/861583ed69a696523b3fb3288736572a2e61f504/gadgetsdk/_gadgetinspectionsession.py#L201) shows the expected calling pattern using the Python `requests` library.

```{.http .apirequest title="Request Body"}
Content-Type: multipart/form-data
```

## Successful Response

```{.json .apiresponse title="Example 200 Response"}
{
    "NextProcessIntervalSec": 40,
    "PrintQuality": 8,
    "WarningSuggested": false,
    "PauseSuggested": false,
    "Score": 5
}
```

| Name                     | Type | Description |
| ------------------------ | :--: | ----------- |
| `NextProcessIntervalSec` | int  | The minimum number of seconds to wait before the next Process API call. The value is at least `20` seconds and may increase based on server load. |
| `PrintQuality`           | int  | A 1-10 print quality score, where `10` is perfect print quality. Use this for user-facing print status. |
| `WarningSuggested`       | bool | `true` when the model is confident the user should be warned about a possible print issue. |
| `PauseSuggested`         | bool | `true` when the model is confident the print has likely failed and should be paused. |
| `Score`                  | int  | A raw 0-100 model score, where `0` is perfect and `100` indicates a strong probability of failure. Use this for advanced processing, not direct user-facing actions. |


## Response Details

### NextProcessIntervalSec

`NextProcessIntervalSec` is the minimum time that must elapse before the next Process API call. Your integration can wait longer, but it must not call the Process API sooner than this interval.

Calling after each minimum interval, or as close as practical, gives the temporal combination model the best data for confident print-state decisions.

### PrintQuality

`PrintQuality` is the user-facing print quality score from the temporal combination model. It ranges from `1` to `10`, where `10` is perfect print quality.

Use `PrintQuality` for UI status, printer displays, dashboards, or other user-facing print health indicators. Do not use this field alone to trigger warnings or pause actions; use `WarningSuggested` and `PauseSuggested` for that.

| Value | Meaning |
| :---: | ------- |
| `1` | Print failure |
| `2` | Probable print failure |
| `3` | Possible print failure |
| `4` | Monitoring a possible print issue |
| `5` | Monitoring a possible print issue |
| `6` | Good print quality |
| `7` | Good print quality |
| `8` | Great print quality |
| `9` | Great print quality |
| `10` | Perfect print quality |

### WarningSuggested

`WarningSuggested` is `true` when the temporal combination model is confident that there may be a print issue and the user should be informed.

The required confidence can be adjusted with `WarningConfidenceLevel` when creating the context. This decision uses several signals, so `PrintQuality` may stay in the `1-3` range for 30-80 seconds before the model has enough confidence to raise this flag.

### PauseSuggested

`PauseSuggested` is `true` when the temporal combination model is confident that the print has probably failed and should be paused.

The required confidence can be adjusted with `PauseConfidenceLevel` when creating the context. Because pausing a print is intrusive, the model waits for high confidence before raising this flag. `PrintQuality` may stay in the `1-3` range for 60-120 seconds before `PauseSuggested` becomes `true`.

### Score

`Score` is the raw temporal combination model score. It ranges from `0` to `100`, where `0` is a perfect print and `100` indicates a strong probability of failure.

Use this value for advanced processing, such as smoothing, aggregation, or custom heuristics. Do not use it directly for user-facing status or actions like warnings and pauses.

## Error Response

If the API does not return a 200 response, it returns an HTTP error code with a common JSON error object.

```{.json .apiresponse title="Example Error Response"}
{
    "ErrorType": "OE_IMAGE_DECODE_FAILED",
    "ErrorDetails": "The uploaded image could not be decoded."
}
```

| Name           | Type   | Description |
| -------------- | :----: | ----------- |
| `ErrorType`    | string | A well-known error type. See [Error Handling](index.md#error-handling). |
| `ErrorDetails` | string | Details about this specific error. |

## Calling Pattern

1. Create a context with the [Create Context API](create-context.md).
2. Send a JPEG snapshot to the `ProcessRequestUrl`.
3. Wait at least `NextProcessIntervalSec`.
4. Send the next snapshot.
5. If the primary process URL fails, switch to `FallbackProcessRequestUrl` for the rest of the context lifetime.
