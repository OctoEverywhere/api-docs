# Webhook Notification Event Types

!!! note
    Remember that over time, more event types will be added to this list.

The following defines the EventType enum:

1. **Print Started**
2. **Print Complete**
3. **Print Failed**
4. **Print Paused**
5. **Print Resumed**
6. **Print Progress**
    - Fired due to a % progress milestone or a timer event.
7. **Gadget Possible Failure Warning**
    - Gadget thinks the print might have a failure.
8. **Gadget Paused Print Due To Failure**
    - Gadget paused the print because it detected a failure.
    - Only fires when Gadget Smart Pause is enabled.
9. **Error**
    - Fired due to any error from the printer.
10. **First Layer Complete**
    - Fired when the first layer of the print is complete.
11. **Filament Change Required**
    - Fired when the printer reports a filament change is required. This can be due to a color swap or because the filament ran out.
12. **User Interaction Required**
    - Fires when the printer requests user interaction.
13. **Non-Supporter Notification Limit**
    - Fired when the user's account hits the daily notification limit.
    - Standard accounts are limited to 3 webhook notifications a day; all supporter roles get unlimited notifications, [learn more here.](https://octoeverywhere.com/supporter?source=web_hook_dev_doc)
14. **Third Layer Complete**
    - Just like FirstLayerComplete, but this fires on the third layer. Some users might find it more useful to check their print after the first few layers are done or both! It's up to you!
15. **Bed Cooldown Complete**
    - Fired when the print bed has cooled down after a print ends.
16. **Test Notification**
    - Fired from the test webhook notification button in the notifications settings page.