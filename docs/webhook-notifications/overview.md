---
title: 3D Printer Webhooks
description: Get free internet or LAN webhook notifications from your 3D printer!
authors:
    - Quinn Damerell
date: 2025-05-20
---

# 🔔 Print Notification Webhooks

[OctoEverywhere's real-time notification engine](https://octoeverywhere.com/notifications?source=devdocs_webhook&handler=webhook) can be extended with custom webhooks. Webhooks allow OctoEverywhere to send instant notifications to any HTTP endpoint you specify.

## What Can You Do With Webhooks?

With a little effort and technical knowledge, your imagination is the limit. You could:

- Play the Mario Kart "race start" sound whenever a print begins.
- Ring a siren and flash lights when [Gadget](https://octoeverywhere.com/gadget) detects a print failure.
- Make your [Emo](https://living.ai/emo/) dance when a print completes successfully.
- Tweet real-time print progress and full-resolution snapshots while a print is running.


!!! question "What Will You Build?"
    [Join our Discord server to share your creations!](https://octoeverywhere.com/r/discord?source=docs_webhooks)

## Setup

See our [Setup Guide](./setup.md) to get started.

## Event Types

See our [Event Types](./event-types.md) page for full details.

## JSON Payload Format

See our [JSON Payload Format](./json-payload-format.md) page for full webhook request body details.


## POST Responses and Constraints

### Return Type

**Your webhook should return HTTP 200 OK.** If your webhook fails too many times in a row, the webhook will be disabled. You can set up the webhook again on the notification page.

### Timeout

**The HTTP request will time out in 10 seconds.** Your endpoint must return a 200 OK before the timeout or the request will be considered a failure.

## Questions?

For issues, questions, or feature requests, reach out to our [development team](https://octoeverywhere.com/support). We would love to hear from you.
