---
title: 3D Printer Webhooks
description: Get free Internet or LAN webhook notifications from your 3D printer!
authors:
    - Quinn Damerell
date: 2025-05-20
---

# 🔔 Print Notification Webhooks

[OctoEverywhere's realtime notification engine](https://octoeverywhere.com/notifications?source=devdocs_webhook&handler=webhook) can be extended with custom webhooks! Webhooks allow OctoEverywhere to send instant notifications to any HTTP endpoint you specify. 

## What Can You Do With WebHooks?

With little effort and technical knowledge, your imagination is the limit! You could...

- Play the Mario Kart "race start" sound anytime you begin a print.
- Ring a siren and flash lights when [Gadget](https://octoeverywhere.com/gadget) detects a print failure.
- Make your [Emo](https://living.ai/emo/) dance when a print completes successfully.
- Tweet realtime print progress and full-resolution snapshots as you're printing.


!!! note
    What can you come up with? [Share your creations on our Discord server!](https://octoeverywhere.com/r/discord?source=docs_webhooks)





### POST Responses And Constraints

**Your webhook should return HTTP 200 OK.** If your webhook fails too many times in a row, the webhook will be disabled. You can setup the webhook again on the notification page.

**The HTTP request will timeout in 10 seconds.** Your endpoint must return a 200 OK before the timeout or the request will be considered a failure.

If you have any issues, questions, or feature requests, reach out to our [development team](https://octoeverywhere.com/support). We would love to hear from you!