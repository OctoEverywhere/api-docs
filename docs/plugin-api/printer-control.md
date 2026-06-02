---
title: Printer Control API
description: OctoEverywhere's printer control API provides printer-agnostic commands for pause, resume, cancel, lights, movement, extrusion, and temperature control.
authors:
    - Quinn Damerell
date: 2026-06-01
---

# Printer Control API

The Printer Control API provides printer-agnostic commands for common actions such as pausing, resuming, canceling, toggling lights, homing, movement, extrusion, and temperature control.

Remember, all plugin APIs share a [common root path](./index.md) and [custom error codes](./../error-codes.md). Check the `Features` bitmask from the [Printer Status API](printer-status.md#feature-flags) before showing optional controls.

!!! tip
    These generic control APIs work with every 3D printer OctoEverywhere supports, including OctoPrint, Moonraker, Klipper, Bambu Lab, Prusa, Elegoo, Creality, and more.

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

Sets bed, chamber, or hotend temperature targets in Celsius. At least one heater target must be supplied. Current safety limits are `75C` for the bed, `75C` for the chamber, and `260C` for tool temperature.

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
| `BedC` | number | No | Bed target temperature in Celsius. Maximum `75C`. |
| `ChamberC` | number | No | Chamber target temperature in Celsius. Maximum `75C`. |
| `ToolC` | number | No | Hotend target temperature in Celsius. Maximum `260C`. |
| `ToolNumber` | int | No | Tool index for multi-tool printers. |


## Common Command Status Codes

| Status | Meaning |
| :----: | ------- |
| `750` | Unknown command failure. |
| `751` | Failed to parse command arguments. |
| `752` | Command execution failed. |
| `753` | Failed to serialize the command response. |
| `754` | Unknown command path. |
| `785` | Host or firmware is not connected. |
| `786` | Invalid printer state for the requested action. |
| `787` | The plugin cannot connect because too many clients are connected. |
| `788` | Feature is not supported on the current platform. |
