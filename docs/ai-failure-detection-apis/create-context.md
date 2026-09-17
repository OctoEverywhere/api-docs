---
title: Create Context API
description: Start an AI failure detection session and get the context and upload URL needed to process print snapshots through Gadget.
og_title: Create AI Failure Detection Contexts
og_description: Initialize a print analysis context, choose model behavior, and get the upload URL your app needs for Gadget-powered failure detection.
authors:
    - Quinn Damerell
date: 2025-05-20
---

# Create Context API

A context is Gadget's analysis session for one print. Create one context for each print, then reuse it for every image from that print.

Contexts expire automatically after 14 days. You do not need to delete them.

!!! tip
    There is no charge for calling this API. AI failure detection API usage pricing only applies to the [Process API](process.md).

Get your [API key](https://octoeverywhere.com/gadgetapi?source=oe_docs_gadget_api_overview_create_context_get_key) without setting up billing. **Free Usage Only** is on by default. See [pricing and free usage](overview.md#pricing) for details.

## HTTP Request

Always send Create Context requests to the primary Gadget API host:

```{.http .apirequest title="HTTP Request"}
POST https://gadget-pv1-oeapi.octoeverywhere.com/api/gadget/v1/createcontext
```

Send `{}` to use the default confidence levels of `3`. In this Bash example, replace `prod_YOUR_API_KEY` with your key:

```bash
curl -X POST \
  'https://gadget-pv1-oeapi.octoeverywhere.com/api/gadget/v1/createcontext' \
  -H 'X-API-Key: prod_YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{}'
```

## Headers

| Name        | Type   | Required | Description |
| ----------- | :----: | :------: | ----------- |
| `X-API-Key` | string | Yes      | Your Gadget API key. |


## JSON Request Body

`WarningConfidenceLevel` and `PauseConfidenceLevel` are optional integers from `1` to `5`. Both default to `3` and control how confident Gadget must be before it suggests a warning or pause. Your app decides how to act on those suggestions.

Lower values make the model more sensitive, which can catch more issues but may increase false positives. Higher values require more confidence, which reduces false positives but may miss smaller failures.

```{.json .apirequest title="Example Request Body"}
{
    "WarningConfidenceLevel": 3,
    "PauseConfidenceLevel": 3
}
```

| Name                     | Type | Default | Description |
| ------------------------ | :--: | :-----: | ----------- |
| `WarningConfidenceLevel` | int  | 3       | Optional. `1` suggests a warning sooner; `5` requires higher confidence before suggesting a warning. |
| `PauseConfidenceLevel`   | int  | 3       | Optional. `1` suggests a pause sooner; `5` requires higher confidence before suggesting a pause. |

## Successful Response

The response contains the context ID and two complete URLs for the [Process API](process.md). Both URLs already include the context ID. Use them as returned; do not rebuild the URL or append the ID.

```{.json .apiresponse title="Example 2XX Response"}
{
    "ContextId": "uYZkEP0g22nPZwtCj6ASWef9Lpmh81Ahs1mnAdw4Z7PlDFbOK6",
    "ProcessRequestUrl": "https://gadget-regional-subdomain.octoeverywhere.com/api/gadget/v1/process/uYZkEP0g22nPZwtCj6ASWef9Lpmh81Ahs1mnAdw4Z7PlDFbOK6",
    "FallbackProcessRequestUrl": "https://gadget-pv1-oeapi.octoeverywhere.com/api/gadget/v1/process/uYZkEP0g22nPZwtCj6ASWef9Lpmh81Ahs1mnAdw4Z7PlDFbOK6"
}
```

| Name                        | Type   | Description |
| --------------------------- | :----: | ----------- |
| `ContextId`                 | string | The ID of the new context. |
| `ProcessRequestUrl`         | string | The complete primary URL. Start sending snapshots for this print here. |
| `FallbackProcessRequestUrl` | string | The complete fallback URL. See [retries and fallback URLs](developer-docs/overview.md#retries-and-fallback-urls) for when to use it. |


## Error Response

If the API does not return a 2XX response, it returns an HTTP error code with a common JSON error object.

```{.json .apiresponse title="Example Error Response"}
{
    "ErrorType": "OE_ARGS_PARSE_FAILED",
    "ErrorDetails": "The request body could not be parsed."
}
```

| Name           | Type   | Description |
| -------------- | :----: | ----------- |
| `ErrorType`    | string | A well-known error type. See [Error Handling](developer-docs/overview.md#error-handling). |
| `ErrorDetails` | string | Details about this specific error. |

For recovery steps, including `OE_API_KEY_IP_RESTRICTED`, see [Error Handling](developer-docs/overview.md#error-handling).

## Next Step

Send a JPEG snapshot to the [Process API](process.md) using the returned `ProcessRequestUrl`, then follow its [inspection timing guidance](process.md#nextprocessintervalsec) for the next image.
