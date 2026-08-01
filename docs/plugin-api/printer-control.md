---
title: 3D Printer Control API
description: Send platform-agnostic commands for starting, pausing, resuming, canceling, lights, movement, extrusion, and temperature control through OctoEverywhere.
og_title: Control 3D Printers Across Platforms
og_description: Add start, pause, resume, cancel, movement, extrusion, lights, and temperature controls to your app without writing separate code for every printer host.
authors:
    - Quinn Damerell
date: 2026-07-31
---

# 3D Printer Control API

The 3D Printer Control API provides printer-agnostic commands for common actions such as starting a file, pausing, resuming, canceling, toggling lights, homing, moving, extruding, and controlling temperature.

!!! tip
    The common API is available on every printer OctoEverywhere supports, but individual controls vary by platform and printer configuration. Check the `Features` bitmask before presenting optional controls and handle status `788` when a feature is unavailable.

[Get Started With Plugin APIs](overview.md){ .md-button .md-button--primary }

## Feature Flags

To find out what features your 3D printer supports, check the `Features` bitmask from the [Printer Status API](printer-status.md#feature-flags) before showing optional controls.

Pause, resume, and cancel are state-dependent core commands and do not have feature bits. The optional controls and start-print command have feature bits documented by the Status API.

## Common Error Codes

All Plugin APIs share a set of [common error codes](./plugin-api-errors.md) that can be returned for issues like the OctoEverywhere plugin is offline, auth issues, etc.

## Response Format

Control APIs return a JSON envelope. `Status` is the plugin command status code. On success, `Status` is `200`.

```{.json .apiresponse title="Example Success Response"}
{
    "Status": 200,
    "Result": {}
}
```

```{.json .apiresponse title="Example Error Response"}
{
    "Status": 786,
    "Error": "Printer state is not printing."
}
```

## Pause

Pauses the current print. If `SmartPause` is enabled, supported platforms can optionally disable heaters, lift Z, retract filament, suppress pause notifications, and show a smart pause popup.

```{.http .apirequest title="HTTP Request"}
POST https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/pause
```

```{.json .apirequest title="Example Request Body"}
{
    "SmartPause": true,
    "DisableHotend": true,
    "DisableBed": false,
    "ZLiftMm": 5,
    "RetractFilamentMm": 2,
    "SuppressNotification": true,
    "ShowSmartPausePopup": true
}
```

| Name | Type | Default | Description |
| ---- | :--: | :-----: | ----------- |
| `SmartPause` | bool | `false` | Enables smart pause behavior when supported. |
| `DisableHotend` | bool | `true` | Smart pause option to turn off the hotend. |
| `DisableBed` | bool | `false` | Smart pause option to turn off the bed. |
| `ZLiftMm` | int | `0` | Smart pause Z lift distance in millimeters. |
| `RetractFilamentMm` | int | `0` | Smart pause filament retract distance in millimeters. |
| `SuppressNotification` | bool | `SmartPause` | Suppresses the OctoEverywhere pause notification. |
| `ShowSmartPausePopup` | bool | `true` | Shows a smart pause popup in the printer portal when relevant. |

Send a JSON object when using pause options. When a body is supplied, `SuppressNotification` defaults to the value of `SmartPause`. For compatibility, a request with no body currently defaults `SuppressNotification` to `true`; set it explicitly when notification behavior matters.


## Resume

Resumes a paused print.

```{.http .apirequest title="HTTP Request"}
POST https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/resume
```

No request body is required.

## Cancel

Cancels the active print.

```{.http .apirequest title="HTTP Request"}
POST https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/cancel
```

No request body is required.

## Start Print

Starts printing a file identified by its OctoEverywhere virtual file path. Check for `FEATURE_PRINT_START` (`32`) before offering this action.

```{.http .apirequest title="HTTP Request"}
POST https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/start
```

```{.json .apirequest title="Example Request Body"}
{
    "Path": "gcode/models/benchy.gcode"
}
```

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `Path` | string | Yes | Virtual file path under the `gcode` root. Use the `VirtualPath` returned by [List Files](files-api.md#list-files) when that platform supports file listing. |
| `Storage` | string | PrusaLink only | PrusaLink storage name. Defaults to `local`. |

For Bambu Lab `.3mf` project files, these additional fields customize the native project-file start request. They are ignored for ordinary G-code files and other platforms.

| Name | Type | Default | Description |
| ---- | :--: | :-----: | ----------- |
| `Plate` | int | `1` | One-based plate number inside the project file. Values below `1` are treated as `1`. |
| `UseAms` | bool | `true` | Enables the AMS for the print. |
| `AmsMapping` | list of int | None | Native Bambu material mapping array. Values normally come from `MaterialSystem.Sources[].PrintMappingValue`. |
| `FlowCali` | bool | `true` | Enables flow calibration. |
| `Timelapse` | bool | Printer default | Overrides timelapse behavior when supplied. |
| `BedLeveling` | bool | Printer default | Overrides bed leveling when supplied. |
| `VibrationCali` | bool | Printer default | Overrides vibration calibration when supplied. |
| `LayerInspect` | bool | Printer default | Overrides layer inspection when supplied. |

The command is currently implemented for OctoPrint, Moonraker/Klipper, Bambu Lab, and PrusaLink. Unsupported platforms return status `788`.

```{.json .apiresponse title="Example Success Response"}
{
    "Status": 200,
    "Result": {
        "VirtualPath": "gcode/models/benchy.gcode",
        "PlatformPath": "models/benchy.gcode",
        "PrinterResponse": {
            "result": "ok"
        }
    }
}
```

`PrinterResponse` is optional and contains the platform-native response when one is available.

## Set Light

Turns a printer light on or off. Use a light `Name` returned by `JobStatus.Lights[]` from the [Printer Status API](printer-status.md#job-status).

```{.http .apirequest title="HTTP Request"}
POST https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/set-light
```

```{.json .apirequest title="Example Request Body"}
{
    "Name": "chamber",
    "On": true
}
```

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `Name` | string | Yes | Light name from `JobStatus.Lights[].Name`. |
| `On` | bool | Yes | `true` turns the light on; `false` turns it off. |


## Move Axis

Moves one axis by a relative distance in millimeters. Positive and negative distances are supported.

```{.http .apirequest title="HTTP Request"}
POST https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/move-axis
```

```{.json .apirequest title="Example Request Body"}
{
    "Axis": "Z",
    "DistanceMm": 5.0
}
```

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `Axis` | string | Yes | Axis to move. Supported values are `X`, `Y`, and `Z`. |
| `DistanceMm` | number | Yes | Relative movement distance in millimeters. |


## Home

Homes all axes.

```{.http .apirequest title="HTTP Request"}
POST https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/home
```

No request body is required.

## Extrude

Extrudes or retracts filament on the selected extruder.

```{.http .apirequest title="HTTP Request"}
POST https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/extrude
```

```{.json .apirequest title="Example Request Body"}
{
    "Extruder": 0,
    "DistanceMm": 10.0
}
```

| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `Extruder` | int | Yes | Zero-based extruder index. |
| `DistanceMm` | number | Yes | Positive values extrude; negative values retract. |


## Set Temp

Sets bed, chamber, or hotend temperature targets in Celsius. At least one heater target must be supplied. Current accepted ranges are `0C` through `75C` for the bed, `0C` through `75C` for the chamber, and `0C` through `260C` for tool temperature. A target of `0` turns that heater off.

```{.http .apirequest title="HTTP Request"}
POST https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/set-temp
```

```{.json .apirequest title="Example Request Body"}
{
    "BedC": 60.0,
    "ToolC": 220.0,
    "ToolNumber": 0
}
```


| Name | Type | Required | Description |
| ---- | :--: | :------: | ----------- |
| `BedC` | number | No | Bed target from `0C` through `75C`. Use `0` to turn it off. |
| `ChamberC` | number | No | Chamber target from `0C` through `75C`. Use `0` to turn it off. |
| `ToolC` | number | No | Hotend target from `0C` through `260C`. Use `0` to turn it off. |
| `ToolNumber` | int | No | Zero-based, non-negative tool index for multi-tool printers. |
