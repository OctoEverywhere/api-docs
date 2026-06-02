---
title: Create Context API
description: Create a new AI failure detection context for a 3D print.
authors:
    - Quinn Damerell
date: 2025-05-20
---

# Create Context API

The Create Context API creates a new AI failure detection context. A context lasts for one print, so create a fresh context for every new print.

Contexts do not need to be destroyed manually. They automatically expire after 14 days.

!!! tip
    There is no charge for calling this API. AI failure detection API usage pricing only applies to the [Process API](process.md).

## HTTP Request

This API should always be called on the primary Gadget API host.

```{.http .apirequest title="HTTP Request"}
POST https://gadget-pv1-oeapi.octoeverywhere.com/api/gadget/v1/createcontext
```

## Headers

| Name        | Type   | Required | Description |
| ----------- | :----: | :------: | ----------- |
| `X-API-Key` | string | Yes      | Your OctoEverywhere developer API key. |


## JSON Request Body

`WarningConfidenceLevel` and `PauseConfidenceLevel` are optional. Both values control how confident the temporal combination model must be before it suggests a warning or pause action.

Lower values make the model more sensitive, which can catch more issues but may increase false positives. Higher values require more confidence, which reduces false positives but may miss smaller failures.

```{.json .apirequest title="Example Request Body"}
{
    "WarningConfidenceLevel": 3,
    "PauseConfidenceLevel": 3
}
```

| Name                     | Type | Default | Description |
| ------------------------ | :--: | :-----: | ----------- |
| `WarningConfidenceLevel` | int  | 3       | Optional. Sets the warning confidence level from `1` to `5`, where `1` warns sooner and `5` warns only with higher confidence. |
| `PauseConfidenceLevel`   | int  | 3       | Optional. Sets the pause confidence level from `1` to `5`, where `1` pauses sooner and `5` pauses only with higher confidence. |

## Successful Response

The response contains the context ID and the two URLs used by the [Process API](process.md).

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
| `ProcessRequestUrl`         | string | The primary URL to use for all Process API calls for the lifetime of this context. |
| `FallbackProcessRequestUrl` | string | The fallback URL to use if `ProcessRequestUrl` fails. Once your app switches to this URL, continue using it for the lifetime of the context. |


## Error Response

If the API does not return a 2XX response, it returns an HTTP error code with a common JSON error object.

```{.json .apiresponse title="Example Error Response"}
{
    "ErrorType": "OE_BAD_ARGS",
    "ErrorDetails": "The request body could not be parsed."
}
```

| Name           | Type   | Description |
| -------------- | :----: | ----------- |
| `ErrorType`    | string | A well-known error type. See [Error Handling](index.md#error-handling). |
| `ErrorDetails` | string | Details about this specific error. |

## Next Step

After creating a context, call the [Process API](process.md) using the returned `ProcessRequestUrl`.
