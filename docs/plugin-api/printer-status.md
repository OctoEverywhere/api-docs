---
title: Printer Status API
description: OctoEverywhere's printer status API returns a printer-agnostic status object for the current print.
authors:
    - Quinn Damerell
date: 2026-06-01
---

# Printer Status API

The Printer Status API returns a printer-agnostic JSON object with the current printer state, active print details, available light states, OctoEverywhere Gadget status, platform version, supported control features, and webcam summary.

Remember, all plugin APIs share a [common root path](./index.md) and [custom error codes](./../error-codes.md).

!!! tip
    This generic status API works with every 3D printer OctoEverywhere supports, including OctoPrint, Moonraker, Klipper, Bambu Lab, Prusa, Elegoo, Creality, and more.

## HTTP Request

```{.http .apirequest title="HTTP Request"}
GET https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/status
```

## Response Format

Plugin command APIs return a JSON envelope. `Status` is the plugin command status code. On success, `Status` is `200` and the payload is in `Result`.

```{.json .apiresponse title="Example 200 Response"}
{
    "Status": 200,
    "Result": {
        "JobStatus": {
            "State": "printing",
            "SubState": null,
            "Error": null,
            "Lights": [
                {
                    "Name": "chamber",
                    "On": true
                }
            ],
            "CurrentPrint": {
                "Progress": 42.5,
                "DurationSec": 3600,
                "TimeLeftSec": 4800,
                "FileName": "example-print",
                "EstTotalFilUsedMm": 12345,
                "EstTotalFilWeightMg": 9876,
                "CurrentLayer": 62,
                "TotalLayers": 145,
                "Temps": {
                    "BedActual": 60.1,
                    "BedTarget": 60.0,
                    "HotendActual": 219.8,
                    "HotendTarget": 220.0,
                    "ChamberActual": 38.2,
                    "ChamberTarget": 40.0
                }
            }
        },
        "OctoEverywhereStatus": {
            "MostRecentPrintIdStr": "a-print-id",
            "PrintStartTimeSec": 1716220800,
            "Gadget": {
                "LastScore": 7.5,
                "ScoreHistory": [7.5, 8.0],
                "TimeSinceLastScoreSec": 35.2,
                "IntervalSec": 40.0,
                "IsSuppressed": false,
                "TimeSinceLastWarnSec": null,
                "TimeSinceLastPauseSec": null
            }
        },
        "PlatformVersion": "1.0.0",
        "Features": 31,
        "ListWebcams": {
            "DefaultIndex": 0,
            "Webcams": []
        }
    }
}
```

Some fields can be `null`, `0`, or omitted when the host platform does not provide that data.

## Job Status

| Name | Type | Description |
| ---- | :--: | ----------- |
| `State` | string | Common printer state. Expected values include `idle`, `warmingup`, `printing`, `paused`, `resuming`, `complete`, `cancelled`, and `error`. |
| `SubState` | string \| null | Optional user-facing detail about the current state, such as a calibration or pause reason. |
| `Error` | string \| null | Optional short user-facing error string. |
| `Lights` | list \| null | Available controllable lights. Use `Lights[].Name` with the [Set Light](printer-control.md#set-light) API. |
| `CurrentPrint` | object | Current print progress, timing, file, layer, filament, and temperature details. |


## Current Print

| Name | Type | Description |
| ---- | :--: | ----------- |
| `Progress` | float | Print progress from `0.0` to `100.0`. |
| `DurationSec` | int | Elapsed print duration in seconds. |
| `TimeLeftSec` | int \| null | Estimated remaining print time in seconds. |
| `FileName` | string | Current or most recent print file name. |
| `EstTotalFilUsedMm` | int | Estimated total filament length for the print, in millimeters. |
| `EstTotalFilWeightMg` | int | Estimated total filament weight for the print, in milligrams, when available. |
| `CurrentLayer` | int \| null | Current layer number, when available. |
| `TotalLayers` | int \| null | Total layer count, when available. |
| `Temps` | object | Temperature readings and targets in Celsius. |


## Temperatures

| Name | Type | Description |
| ---- | :--: | ----------- |
| `BedActual` | float | Current bed temperature in Celsius. |
| `BedTarget` | float | Target bed temperature in Celsius. |
| `HotendActual` | float | Current hotend temperature in Celsius. |
| `HotendTarget` | float | Target hotend temperature in Celsius. |
| `ChamberActual` | float | Current chamber temperature in Celsius, when available. |
| `ChamberTarget` | float | Target chamber temperature in Celsius, when available. |


## Feature Flags

`Features` is a bitmask that tells you which [Printer Control APIs](printer-control.md) are supported by the current platform.

| Flag | Value | Control API |
| :--: | :---: | ----------- |
| `FEATURE_LIGHT_CONTROL` | `1` | [Set Light](printer-control.md#set-light) |
| `FEATURE_AXIS_MOVEMENT` | `2` | [Move Axis](printer-control.md#move-axis) |
| `FEATURE_HOMING` | `4` | [Home](printer-control.md#home) |
| `FEATURE_EXTRUSION` | `8` | [Extrude](printer-control.md#extrude) |
| `FEATURE_TEMPERATURE_CONTROL` | `16` | [Set Temp](printer-control.md#set-temp) |

## Error Response

If the plugin cannot read printer status, the command still returns a JSON envelope with a non-`200` `Status`.

```{.json .apiresponse title="Example Error Response"}
{
    "Status": 785,
    "Error": "Host not connected"
}
```
