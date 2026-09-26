---
title: Process API
description: Send JPEG print snapshots to OctoEverywhere AI and get real-time failure signals, confidence, warning state, and status details.
og_title: Process 3D Print Snapshots with Gadget AI
og_description: Send webcam images to OctoEverywhere and get actionable print-quality signals your app can use to warn users or pause risky prints.
authors:
    - Quinn Damerell
date: 2025-05-20
---

# Process API

Send a JPEG snapshot and get a print-quality score, warning and pause suggestions, and the minimum and recommended delays before your next snapshot.

Start with the [Create Context API](create-context.md). Using the same context lets Gadget consider results from earlier snapshots when checking how the print is progressing.

!!! tip
    Get your [API key](https://octoeverywhere.com/gadgetapi?source=oe_docs_gadget_api_overview_process_get_key) without setting up billing. See [pricing and free usage](overview.md#pricing) for details.

## HTTP Request

Use the full `ProcessRequestUrl` returned by the [Create Context API](create-context.md), exactly as returned. It already includes the context ID. See [retries and fallback URLs](developer-docs/overview.md#retries-and-fallback-urls) if that server is unavailable.

```{.http .apirequest title="HTTP Request"}
POST https://<your-process-host>.octoeverywhere.com/api/gadget/v1/process/{ID}
```

## Headers


| Name        | Type   | Required | Description |
| ----------- | :----: | :------: | ----------- |
| `X-API-Key` | string | Yes      | Your Gadget API key. |


## Path Parameters

| Name | Type   | Required | Description |
| ---- | :----: | :------: | ----------- |
| `ID` | string | Yes      | The context ID returned by the [Create Context API](create-context.md). |


## Request Body

Send exactly one image file in a `multipart/form-data` POST body; JPEG is recommended. The image file can be up to **6 MiB (6,291,456 bytes)**, excluding multipart overhead. The field name and filename can be anything; this example uses `image` and `print.jpg`.

### Example Request

Replace the URL placeholder with the full `ProcessRequestUrl`, add your key, and point curl at your JPEG file:

```{.bash .apirequest title="curl Example"}
PROCESS_REQUEST_URL='PASTE_FULL_ProcessRequestUrl_HERE'

curl --request POST "$PROCESS_REQUEST_URL" \
    -H 'X-API-Key: prod_YOUR_API_KEY' \
    -F 'image=@print.jpg'
```

Let curl set `Content-Type` so it includes the required multipart boundary. For Python, the [standalone HTTP example](https://github.com/OctoEverywhere/Gadget-Python-Sdk/blob/main/examples/raw_requests.py) shows the upload, timing, and error handling using `requests`.

## Successful Response

```{.json .apiresponse title="Example 200 Response"}
{
    "NextProcessIntervalSec": {
        "Minimum": 5,
        "Recommended": 20
    },
    "FasterInspectionSuggested": false,
    "PrintQuality": 8,
    "WarningSuggested": false,
    "PauseSuggested": false,
    "Score": 5
}
```

### Which fields should I use?

| Name                     | Type | Use it for |
| ------------------------ | :--: | ----------- |
| `NextProcessIntervalSec` | object | Scheduling the next snapshot. Contains `Minimum` and `Recommended` delays in seconds; use the latest response. |
| `FasterInspectionSuggested` | bool | Optionally switching from a fixed interval to the recommended interval while Gadget builds confidence about a possible issue. Treat a missing field as `false`. |
| `PrintQuality`           | int  | Showing print status in your UI. Ranges from `1` to `10`, with `10` best. |
| `WarningSuggested`       | bool | Deciding when to warn the user about a possible print issue. |
| `PauseSuggested`         | bool | Deciding when to pause a print that has likely failed. |
| `Score`                  | int  | Advanced analysis. Most apps can ignore this raw `0`-`100` score; `0` is best, the opposite of `PrintQuality`. |

The warning and pause flags are recommendations. Your software sends the warning or pauses the printer.

## Response Details

### NextProcessIntervalSec

Use **`NextProcessIntervalSec.Recommended` as the default inspection interval**, taking the value from the latest Process API response. You can inspect faster than recommended, provided you wait at least `NextProcessIntervalSec.Minimum` seconds between calls.

| Field | Type | Description |
| ----- | :--: | ----------- |
| `Minimum` | int | The shortest allowed delay in seconds before the next snapshot. Currently `5`. |
| `Recommended` | int | The server's suggested delay in seconds before the next snapshot. Use this by default. |

For a custom interval, wait `max(your_configured_interval, NextProcessIntervalSec.Minimum)` seconds before sending the next snapshot. A fixed interval is supported for predictable usage. You can optionally follow [`FasterInspectionSuggested`](#fasterinspectionsuggested) to temporarily use the recommended interval when Gadget detects a possible issue. If no custom interval is configured, use `NextProcessIntervalSec.Recommended`.

All [print-hour pricing](overview.md#pricing) uses a 20-second interval: 180 inspection calls equal one print hour. Billing is based on inspection calls, so inspecting faster uses more calls, consumes the free allowance sooner, and increases paid usage costs. A 5-second interval uses four times as many calls per actual hour of printing as a 20-second interval.

### FasterInspectionSuggested

`FasterInspectionSuggested` is an **optional timing hint** for apps that normally use a fixed inspection interval. You can ignore it and keep your configured interval for predictable usage, or use it to follow the same signal that makes OctoEverywhere inspect more frequently internally. Apps that already follow `NextProcessIntervalSec.Recommended` need no extra timing logic.

The flag is normally `false` and is expected to stay off roughly 99% of the time during normal printing; this is an expectation, not a guaranteed percentage. Gadget sets it to `true` only when it has strong evidence of a possible issue and wants faster snapshots to build confidence before recommending a warning or pause. It covers possible warning and pause conditions, and can still be `true` on the response that first sets `WarningSuggested` or `PauseSuggested`.

To opt in, choose your next delay after each successful response:

- If `FasterInspectionSuggested` is `true`, use the latest `NextProcessIntervalSec.Recommended`.
- If it is `false` or missing, return to your configured interval. Treating a missing field as `false` keeps clients compatible with older responses.
- **Always wait at least the latest `NextProcessIntervalSec.Minimum`**, currently 5 seconds. This flag never permits calls below the minimum.

For example, with a configured interval in seconds:

```python
intervals = response["NextProcessIntervalSec"]
next_delay = configured_interval
if response.get("FasterInspectionSuggested", False):
    next_delay = intervals["Recommended"]
next_delay = max(next_delay, intervals["Minimum"])
```

Use `WarningSuggested` and `PauseSuggested` to decide when to act; `FasterInspectionSuggested` only controls timing. Temporarily shortening the interval makes more inspection calls, which can consume the free allowance sooner and increase paid usage costs.

### PrintQuality

`PrintQuality` ranges from `1` to `10`, where `10` is perfect print quality.

Use it on printer displays, dashboards, or anywhere you show print status. For warnings and pause actions, use `WarningSuggested` and `PauseSuggested` instead of acting on this score alone.

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

`WarningSuggested` is `true` when Gadget is confident enough to recommend warning the user about a possible print issue.

Adjust the required confidence with `WarningConfidenceLevel` when creating the context. Gadget considers several signals over time, so `PrintQuality` may stay in the `1-3` range for 30-80 seconds before this flag becomes `true`.

### PauseSuggested

`PauseSuggested` is `true` when Gadget is confident enough to recommend pausing a print that has probably failed.

The required confidence can be adjusted with `PauseConfidenceLevel` when creating the context. Because pausing a print is intrusive, the model waits for high confidence before raising this flag. `PrintQuality` may stay in the `1-3` range for 60-120 seconds before `PauseSuggested` becomes `true`.

### Score

Most apps can ignore `Score`. It's the raw model score, from `0` for a perfect print to `100` for a strong probability of failure.

Use it for custom analysis, such as smoothing or combining results. Use the quality score and suggestion flags for user-facing status and actions.

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
| `ErrorType`    | string | A well-known error type. See [Error Handling](developer-docs/overview.md#error-handling). |
| `ErrorDetails` | string | Details about this specific error. |

### API Key IP Restricted

HTTP `403` with `ErrorType` set to `OE_API_KEY_IP_RESTRICTED` means another API key has already claimed this request's public IP address. Stop sending inspections with the rejected key and follow the recovery guidance in [Error Handling](developer-docs/overview.md#error-handling). Do not retry automatically or switch to the fallback URL for this error.

### Free Usage Limit Reached

HTTP `429` with `ErrorType` set to `OE_FREE_USAGE_LIMIT_REACHED` means the monthly free allowance has been exhausted and either billing is not set up or **Free Usage Only** is enabled.

Stop sending inspections until the allowance resets, or until the account owner sets up billing and turns off **Free Usage Only** at [OctoEverywhere.com/gadgetapi](https://octoeverywhere.com/gadgetapi?source=oe_docs_gadget_api_overview_process_free_limit). Creating a new context or switching to the fallback URL does not reset the allowance.

Handle this error separately from temporary rate limiting. Do not retry it in a loop; a `Retry-After` header does not mean the monthly allowance has reset.

## Calling Pattern

After each successful request, wait for your [inspection interval](#nextprocessintervalsec), then send the next snapshot. See [retries and fallback URLs](developer-docs/overview.md#retries-and-fallback-urls) and [error handling](developer-docs/overview.md#error-handling) for recovery steps.
