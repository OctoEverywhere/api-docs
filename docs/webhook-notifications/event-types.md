---
title: Webhook Notification Event Types
description: Review every OctoEverywhere webhook event type your endpoint can receive, from print progress and completion to Gadget AI warnings and printer errors.
og_title: Know Every OctoEverywhere Webhook Event
og_description: Map OctoEverywhere EventType values to real printer events so your app can route print starts, completions, progress, failures, Gadget alerts, and errors.
---

# Webhook Notification Event Types

## Overview

These Event Types are common to both [Notification webhooks](./overview.md) and [App Connection Webhooks](../app-connections/apis/notification-webhook.md).

!!! note
    These Event Type values will never be removed or re-arranged, but new Event Types will be added over time.

## Event Type Enum Definition

1. **Print Started**
2. **Print Complete**
3. **Print Failed**
4. **Print Paused**
5. **Print Resumed**
6. **Print Progress**
    - Fired for two reasons:
        - Fired incrementally for every 10% of print progress.
        - AND fired incrementally for every hour of print progress.
7. **Gadget Possible Failure Warning**
    - Gadget thinks the print might have a failure.
8. **Gadget Paused Print Due to Failure**
    - Gadget paused the print because it detected a failure.
    - Only fires when Gadget Smart Pause is enabled.
9. **Error**
    - Fired due to any error from the printer. The payload will have a platform-specific error message and platform error code.
10. **First Layer Complete**
    - Fired when the first layer of the print is complete.
11. **Filament Change Required**
    - Fired when the printer reports a filament change is required. This can be due to a color swap or because the filament ran out.
12. **User Interaction Required**
    - Fires when the printer requests user interaction. The payload will have a platform-specific error message and platform error code.
13. **Non-Supporter Notification Limit**
    - Fired when the user's account reaches the daily notification limit.
    - Standard accounts are limited to 3 webhook notifications a day; all supporter roles get unlimited notifications. [Learn more.](https://octoeverywhere.com/supporter?source=web_hook_dev_doc)
14. **Third Layer Complete**
    - Similar to FirstLayerComplete, but this fires on the third layer. Some users may prefer to check a print after the first few layers, or after both the first and third layers. It's up to you.
15. **Bed Cool Down Complete**
    - Fired when the print bed has cooled down after a print ends.
16. **Test Notification**
    - Fired from the test webhook notification button in the notifications settings page or by using the [App Connection Webhook Test API](../app-connections/apis/notification-webhook.md).
