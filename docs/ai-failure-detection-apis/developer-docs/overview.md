---
title: Gadget API Developer Overview
description: Add Gadget AI print failure detection to your app with a Python SDK, simple API examples, and guides to handling results, errors, and retries.
og_title: Gadget AI Failure Detection Developer Docs
og_description: Create a print context, send webcam snapshots, and use Gadget's results to warn users or pause a print.
---

# Gadget API Developer Overview

Add Gadget to your app in three steps: create a context for a print, send webcam snapshots, and use the results to warn the user or pause the print. Start a new context for each print.

For API keys, pricing, free usage, and privacy, see the [Gadget API overview](../overview.md).

## API Overview

Include your API key in the `X-API-Key` header on both API calls.

### 1. Create a Print Context

A context keeps track of one print. Call [Create Context](../create-context.md) when the print starts, then reuse that context for all snapshots from that print. Creating a context is free.

The response includes:

- `ContextId`: the ID for this print's context.
- `ProcessRequestUrl`: the full URL to send snapshots to. It already includes the context ID.
- `FallbackProcessRequestUrl`: a backup URL for the same context. Keep it for [connection or server failures](#retries-and-fallback-urls).

### 2. Send a Snapshot

Send one JPEG image to `ProcessRequestUrl` as a `multipart/form-data` POST request. The [Process API](../process.md#example-request) has a complete upload example.

Gadget's temporal model combines the latest image analysis with earlier results from the print. This lets it follow how the print develops over time.

Use `PrintQuality` to display print status, and `WarningSuggested` or `PauseSuggested` to decide when to warn the user or pause the print. Your software takes those actions. See [Which Fields Should I Use?](../process.md#which-fields-should-i-use) for a quick guide to the response.

### 3. Wait and Repeat

Use **`NextProcessIntervalSec.Recommended` as the default interval** from the latest response. To inspect faster, choose a custom interval that is at least `NextProcessIntervalSec.Minimum`, currently 5 seconds. Faster inspections use more calls and increase paid usage costs.

For predictable usage, you can keep a fixed interval. Optionally follow [`FasterInspectionSuggested`](../process.md#fasterinspectionsuggested): when it is `true`, use the latest `NextProcessIntervalSec.Recommended`; when it is `false` or missing, return to your configured interval. Always respect `NextProcessIntervalSec.Minimum`. This flag is normally off and requests faster snapshots only when Gadget has strong evidence of a possible issue and needs to build confidence. Continue using `WarningSuggested` and `PauseSuggested` for actions.

Keep sending snapshots while the print is running. See [inspection timing](../process.md#nextprocessintervalsec) for custom intervals and how they affect usage.

## SDKs

Using Python? Start with our [Gadget Python SDK](https://github.com/OctoEverywhere/Gadget-Python-Sdk).

You can also call the APIs directly from your preferred language. The endpoint pages include request examples, and the [standalone Python HTTP example](https://github.com/OctoEverywhere/Gadget-Python-Sdk/blob/main/examples/raw_requests.py) shows the full flow without depending on the SDK.

## Error Handling

API errors return a non-200 HTTP status and a JSON body:

```json
{
    "ErrorType": "OE_INTERNAL_ERROR",
    "ErrorDetails": "A string with error details."
}
```

Use `ErrorType` to decide what to do. `ErrorDetails` explains the specific problem.

### Retry After a Delay

Wait until your next inspection interval before retrying. Avoid immediate retry loops.

| Error | Meaning | What to Do |
| --- | --- | --- |
| `OE_CONTEXT_RATE_LIMITED` | HTTP `429`: this context is sending snapshots too quickly. | Increase the delay and respect the latest `NextProcessIntervalSec.Minimum`. |
| `OE_BACKEND_THROTTLED` | Gadget is temporarily busy. | Wait before sending the next snapshot. |
| `OE_INTERNAL_ERROR` | Gadget couldn't complete the request. | Wait and retry. See [Retries and Fallback URLs](#retries-and-fallback-urls) if the server is unavailable. |

### Fix the Request or Account

| Error | Meaning | What to Do |
| --- | --- | --- |
| `OE_BAD_ARGS` | A request value is missing or invalid. | Check `ErrorDetails` and fix the request. |
| `OE_ARGS_PARSE_FAILED` | The JSON request body couldn't be read. | Check that the body is valid JSON. |
| `OE_IMAGE_DECODE_FAILED` | Gadget couldn't decode the image. | Send a valid JPEG. |
| `OE_INVALID_API_KEY` | The API key is missing or incorrect. | Check the `X-API-Key` header. |
| `OE_API_KEY_DISABLED` | The API key has been disabled. | [Contact our team](https://octoeverywhere.com/support?instant=t&source=oe_docs_gadget_api_overview_error_disabled) to restore access. |
| `OE_API_KEY_BLOCKED_PAYMENT_FAILED` | The account's billing needs attention. | Ask the account owner to check their [billing settings](../overview.md#stay-free-or-enable-paid-usage). |
| `OE_API_KEY_IP_RESTRICTED` | HTTP `403`: another API key has already claimed this public IP address. | Stop retrying and use the original key. See [IP restrictions](#ip-restrictions) below. |
| `OE_FREE_USAGE_LIMIT_REACHED` | HTTP `429`: the free allowance is used up while billing isn't set up or **Free Usage Only** is on. | Stop inspections until the next billing period, or [enable paid usage](#free-usage-limit). |

!!! tip "Not every 429 means retry"
    Check `ErrorType`. A rate-limited context needs a longer delay. An exhausted free allowance needs a new billing period or paid usage. A short `Retry-After` header does not mean the monthly allowance has reset.

### Free Usage Limit

The free allowance applies across the account. Creating another context or switching processing URLs does not reset it.

To continue with paid usage, the account owner can [set up billing](https://octoeverywhere.com/gadgetapi?source=oe_docs_gadget_api_overview_error_free_limit) and turn off **Free Usage Only**. If billing is already set up, they only need to turn off that setting. Otherwise, wait for the next monthly allowance.

### IP Restrictions

The first successfully authenticated API key used from a public IP address claims that IP. Other keys cannot use it, including keys from accounts sharing the same public IP. The same key can still use multiple IPs.

Use your original key, or [contact our team](https://octoeverywhere.com/support?instant=t&source=oe_docs_gadget_api_overview_error_ip_restricted) if that key is unavailable or the IP is shared with another account. Creating another key, setting up billing, or changing **Free Usage Only** does not remove the restriction.

## Retries and Fallback URLs

Start with `ProcessRequestUrl`. If a connection or server failure prevents processing, switch to `FallbackProcessRequestUrl` and keep using it for the rest of that print's context. The fallback can route requests to another region.

When you receive a JSON API error, check [Error Handling](#error-handling) first. Invalid requests, account problems, IP restrictions, and usage limits need the actions listed there; switching URLs won't fix them.

Keep the same context when retrying or switching URLs, and wait between attempts. Use the most recent `NextProcessIntervalSec.Recommended` by default, or a custom inspection interval that is at least `NextProcessIntervalSec.Minimum`. Before the first successful response, use 20 seconds as the default delay between attempts.

