# 🤖 AI Print Failure Detection APIs

OctoEverywhere empowers the 3D printing community with free and unlimited AI print failure detection through [Gadget](https://octoeverywhere.com/gadget?source=dev_docs_ai). Gadget processes millions of images and hundreds of gigabytes of print data every day.

Our AI failure detection service uses advanced ML models to process webcam images with high accuracy. Those models are continuously improved with real-time community feedback, making them more adaptive, robust, and accurate over time.

To turn image analysis into useful print decisions, OctoEverywhere also uses a temporal combination model. This model evaluates several signals over time to decide when to warn a user about a possible issue or suggest pausing a print.

Together, these systems run on OctoEverywhere's high-availability global server network and power the same failure detection experience trusted by the OctoEverywhere community.

!!! tip
    **You do not need to attribute your AI failure detection system to OctoEverywhere.** You're free to build whatever you want.

## What You Can Build

OctoEverywhere AI Failure Detection APIs give you access to the same AI failure detection service used by OctoEverywhere. The API is simple to integrate: your app or service sends JPEG images to OctoEverywhere, and the service returns the print quality signals needed to display status, warn users, or take action.

## Pricing

You only pay for what you use. **Every developer gets 5,000 API calls per month for free, which is about 55 hours of AI failure detection.**

Pricing uses additive tiers, so the cost per call decreases as your usage grows.

| Number of API Calls | Price Per API Call in USD |
| ------------------- | ------------------------- |
| 0-5,000             | Free                      |
| 5,000-1m            | $0.0004                   |
| 1m-20m              | $0.00025                  |
| 20m+                | $0.00015                  |

We offer volume pricing for customers with high API demand. [Contact us to discuss details.](https://octoeverywhere.com/support?source=dev_docs_ai_failure_detection)

## SDKs

We currently offer an [AI Failure Detection SDK for Python](https://github.com/OctoEverywhere/Gadget-Python-Sdk), with more languages coming later.

**We recommend using the [Python SDK](https://github.com/OctoEverywhere/Gadget-Python-Sdk/blob/main/gadgetsdk/_gadgetinspectionsession.py) as a reference implementation.** It is the best way to understand the API parameters, calling patterns, and end-to-end flow.

## API Overview

### 1. Create a Print Context

Create one context per print so the ML models have a clean session to work with. There is no charge for this API call.

Create a context by calling the [Create Context API](create-context.md) with your API key and optional confidence parameters. The response includes:

- `ContextId`: the new context ID.
- `ProcessRequestUrl`: the primary URL to use for future [Process API](process.md) calls.
- `FallbackProcessRequestUrl`: the fallback URL to use if the primary URL fails.

Use the primary processing URL unless it fails. If it does, switch to the fallback URL and keep using it for the rest of the context lifetime.

### 2. Process Images

To start AI failure detection, send a JPEG image to the [Process API](process.md) using the `ProcessRequestUrl` returned by [Create Context](create-context.md).

The Process API accepts `multipart/form-data` POST requests. Multipart form data is supported by built-in or standard HTTP libraries in Python, C#, Java, JavaScript, C++, and most other modern languages.

The Process API evaluates the new image against the previous context. The temporal combination model then returns output based on the current image, previous context, and other signals:

- `NextProcessIntervalSec` - The minimum number of seconds to wait before the next Process API call. This value changes based on server load and averages around 40 seconds.
- `PrintQuality` - A 1-10 print quality score, where 10 is perfect print quality. Use this value for user-facing print status.
- `WarningSuggested` - `true` when the model is confident that the user should be warned about a possible print issue.
- `PauseSuggested` - `true` when the model is confident that the print has likely failed and should be paused.
- `Score` - A raw 0-100 model score, where 0 is perfect and 100 indicates a strong probability of failure. Use this for advanced processing, not direct user-facing actions.

### 3. Repeat

Continue calling the [Process API](process.md) with new snapshots to keep the AI model updated with the current print state.

Your integration **must wait at least the number of seconds returned by `NextProcessIntervalSec`** before calling the Process API again. You may wait longer if desired. Calling close to the minimum interval gives the temporal model more signals and can improve failure detection speed and accuracy.

## Error Handling

If any AI failure detection API fails, it returns a non-200 HTTP status code with a common JSON error object:

```json
{
    "ErrorType": "OE_INTERNAL_ERROR",
    "ErrorDetails": "A string with error details."
}
```

`ErrorType` is one of the well-known error strings below. `ErrorDetails` provides additional information about the specific failure.

### Client-Actionable Errors

- `OE_BAD_ARGS` - Required API arguments were missing or invalid.
- `OE_ARGS_PARSE_FAILED` - The API handler failed to parse the JSON request body.
- `OE_CONTEXT_RATE_LIMITED` - The context made too many requests and was rate-limited.
- `OE_IMAGE_DECODE_FAILED` - Returned by the [Process API](process.md) when the JPEG image could not be decoded.
- `OE_INVALID_API_KEY` - The API key in the `X-API-Key` header was missing or incorrect.
- `OE_API_KEY_DISABLED` - The API key has been disabled by the developer account. A disabled key will not be re-enabled; create a new key instead.
- `OE_API_KEY_BLOCKED_PAYMENT_FAILED` - The API key account owner has an outstanding balance that must be paid before the [Process API](process.md) can be used.

### Backend Server Errors

- `OE_BACKEND_THROTTLED` - The [Process API](process.md) request was temporarily throttled. Try again after the next delay interval.
- `OE_INTERNAL_ERROR` - A generic service error. Try again after the next delay interval.

## High Availability

OctoEverywhere's AI failure detection service is designed for high availability by controlling Process API request rate and balancing traffic across regions.

### ProcessRequestUrl and FallbackProcessRequestUrl

`ProcessRequestUrl` and `FallbackProcessRequestUrl` are returned by the [Create Context API](create-context.md) and should be used for the lifetime of the context.

Use `ProcessRequestUrl` for all [Process API](process.md) calls unless a call fails. If a network, server, or other failure occurs, switch to `FallbackProcessRequestUrl` for the rest of the context lifetime.

The primary URL lets the service route traffic intelligently while the system is healthy. The fallback URL uses global routing so requests can be handled by another region if the primary path fails.

Load balancing prioritizes server capacity over physical distance. Process API calls already have meaningful compute latency, so additional network transit time is usually trivial.

### NextProcessIntervalSec

`NextProcessIntervalSec` globally controls how quickly clients can send Process API requests. The goal is to accept as many requests as possible for strong failure detection without overwhelming the service during peak load.

Your app or service may call the [Process API](process.md) at any interval longer than the last returned `NextProcessIntervalSec`. Shorter intervals, as long as they respect that minimum, can produce faster and more accurate failure detection, but they also increase API usage.

## Get in Touch

We would love to hear what you're building and help you get started.

[Contact Our Dev Team](https://octoeverywhere.com/support?instant=t){ .md-button .md-button--primary }
