# Webhook Json Payload Format

!!! note
    Bookmark this page, because over time we will add new notification types and json properties.

The HTTP POST will have a json body defined as:

- **PrinterId**
    - (string) This is a unique ID that defines your printer. This ID will not change as long as your printer is attached to your account.
- **SecretKey**
    - (string) If you set a secret key, it will be included here.
- **PrintId**
    - (string) A string that's unique for each print. This string is created when a print starts and remains until the print is complete or stopped. This is useful for tracking which notifications are associated with which print jobs.
- **EventType**
    - (int, enum) This enum maps to the notification type. [The enum is defined here.](event-types.md)
- **PrinterName**
    - (string) The name you assigned your printer. This name will change if the printer is renamed on OctoEverywhere.
- **SnapshotUrl**
	- (string, optional) If a snapshot can be taken, this will be a URL where the image can be viewed or downloaded. Note this image URL will only remain valid for about 7 days.
- **QuickViewUrl**
    - (string) A url to OctoEverywhere's Quick View, which provides a secure internet based way to quickly view the full printer state, pause, and cancel prints.
- **FileName**
	- (string, optional) the file name in OctoPrint for the current file being printed.
- **DurationSec**
	- (int, optional) the duration of the print, since the start, in seconds.
- **Progress**
	- (int, optional) the current print progress as a percentage. Where 0 is 0% and 100 is 100%.
- **TimeRemainingSec**
	- (int, optional) the amount of time estimated to be remaining by OctoPrint, in seconds.
- **ZOffsetMM**
	- (int, optional) the current z-axis offset in millimeters.
- **Error**
	- (string, optional) for EventTypes that indicate an error, this string might contain some kind of error message from OctoPrint describing the issue.

An example json body might be like this:

```
{
    "PrinterId": "Id",
    "SecretKey": "Key",
    "PrintId": "Id",
    "EventType": 1,
    "PrinterName": "Ender3",
    "SnapshotUrl": "https://...",
    "QuickViewUrl": "https://...",
    "FileName": "MyCoolPrint",
    "DurationSec": 50,
    "Progress": 21,
    "TimeRemainingSec": 605,
    "ZOffsetMM": 300,
}
```