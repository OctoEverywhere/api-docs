---
title: 3D Printer Status API
description: Get real-time printer and print status from OctoPrint, Moonraker, Bambu Lab, PrusaLink, Elegoo, and more through one common API.
og_title: Read Real-Time 3D Printer Status Anywhere
og_description: Use OctoEverywhere's Status API to show printer state, active print progress, temperatures, and platform-agnostic status in your app.
authors:
    - Quinn Damerell
date: 2026-06-01
---

# 3D Printer Status API

The Printer Status API returns a printer-agnostic JSON object with the current printer state, active print details, available light states, OctoEverywhere Gadget status, platform version, supported control features, and webcam summary.

!!! tip
    This common API works with every 3D printer OctoEverywhere supports, including OctoPrint, Moonraker, Klipper, Bambu Lab, Prusa, Elegoo, Creality, and more.

[Get Started With Plugin APIs](overview.md){ .md-button .md-button--primary }


## Common Error Codes

All Plugin APIs share a set of [common error codes](./plugin-api-errors.md) that can be returned for issues like the OctoEverywhere plugin is offline, auth issues, etc.

## HTTP Request

```{.http .apirequest title="HTTP Request"}
GET https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/status
```

### Include Multi-Material And Multi-Tool Data

Material-system data is disabled by default because a printer with many slots or tool heads can produce a much larger response. Set the optional `IncludeMaterialSystem` query parameter to opt in.

```{.http .apirequest title="HTTP Request With Material System"}
GET https://<unique_id>.octoeverywhere.com/octoeverywhere-command-api/status?IncludeMaterialSystem=true
```

| Name | Type | Default | Description |
| ---- | :--: | :-----: | ----------- |
| `IncludeMaterialSystem` | bool | `false` | When `true` or `1`, requests `JobStatus.MaterialSystem`. When `false`, `0`, or omitted, `MaterialSystem` is omitted from the response. |

The aliases `includeMaterialSystem` and `include_material_system` are also accepted. Callers sending JSON command arguments should send a JSON boolean rather than the strings `"true"` or `"false"`.

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


## Material System

When `IncludeMaterialSystem=true`, the response contains `Result.JobStatus.MaterialSystem`.

- An object means the plugin returned normalized material and tool topology.
- `null` means the plugin supports the flag, but the printer platform could not report material-system data.
- The field is omitted when the flag is false or absent. Older plugin versions that do not support the flag may also omit it.

The model deliberately separates where filament comes from and what extrudes it:

| Generic object | What it represents | Platform-specific examples |
| -------------- | ------------------ | -------------------------- |
| `Capabilities` | Independent support flags for multi-material, multi-tool, routing, and drying features. | A tool changer can support multiple tools without supporting switchable material routing. |
| `Sources` | Selectable places where filament originates. | Bambu AMS trays, an external spool, a U1 filament feed, or filament attached to a tool head. |
| `Tools` | Physical extrusion endpoints. | A single hotend, a Bambu dual-nozzle extruder, a U1 extruder, or a Prusa XL tool head. |
| `Units` | Devices or containers that group sources. | An AMS enclosure or another multi-spool feeder. Standalone sources have no unit. |
| `Routes` | Known links from sources to tools. | A loaded AMS tray feeding one nozzle, a source currently loading, or a fixed source-to-tool assignment. |
| `PlatformDetails` | Optional non-portable fields that do not have a safe common mapping. | Bambu tray/RFID data or Snapmaker U1 feed and entanglement-sensor state. |

An AMS can have many sources feeding one tool, while a tool changer can have one source per tool. Hybrid printers can have both. Use `Routes[].SourceId` and `Routes[].ToolId` to join them; never infer a relationship from matching `Index` or `Position` values.

Most optional fields are omitted when unknown. An omitted boolean does not mean `false`, an omitted measurement does not mean zero, and an omitted `Material` object does not mean the source is empty.

```{.json .apiresponse title="Example MaterialSystem Object"}
{
    "Capabilities": {
        "SupportsMultiMaterial": true,
        "SupportsMultiTool": false,
        "SupportsSourceRouting": true,
        "SupportsDrying": false
    },
    "Sources": [
        {
            "Id": "bambu-ams-0-source-0",
            "Index": 0,
            "Name": "AMS 1 Slot 1",
            "UnitId": "bambu-ams-0",
            "Position": 0,
            "PrintMappingValue": 0,
            "IsEmpty": false,
            "Material": {
                "Type": "PLA",
                "Name": "PLA Basic",
                "ColorHex": "FF6600",
                "RemainingPercent": 73.0,
                "Manufacturer": "Bambu Lab"
            },
            "PlatformDetails": {
                "TrayId": 0,
                "TagUid": "example-rfid-tag"
            }
        }
    ],
    "Tools": [
        {
            "Id": "tool-0",
            "Index": 0,
            "Name": "Nozzle",
            "IsActive": true,
            "IsPresent": true,
            "ActualCelsius": 219.8,
            "TargetCelsius": 220.0,
            "NozzleDiameterMm": 0.4,
            "PlatformDetails": {
                "NozzleType": "hardened_steel"
            }
        }
    ],
    "Units": [
        {
            "Id": "bambu-ams-0",
            "Index": 0,
            "Name": "AMS 1",
            "HumidityPercent": 36.0,
            "TemperatureCelsius": 25.4,
            "IsDrying": false,
            "PlatformDetails": {
                "AmsId": "0",
                "HumidityLevel": 3
            }
        }
    ],
    "Routes": [
        {
            "SourceId": "bambu-ams-0-source-0",
            "ToolId": "tool-0",
            "State": "loaded"
        }
    ],
    "PlatformDetails": {
        "AmsVersion": 2
    }
}
```

### Material-System Capabilities

Capability fields describe independent concepts and can be omitted when the platform cannot determine them.

| Name | Type | Description |
| ---- | :--: | ----------- |
| `SupportsMultiMaterial` | bool | The printer can select among multiple material sources, regardless of tool count. |
| `SupportsMultiTool` | bool | The printer has multiple independently selectable extrusion tools, regardless of source count. |
| `SupportsSourceRouting` | bool | The platform reports meaningful source-to-tool routing instead of only fixed or inferred links. |
| `SupportsDrying` | bool | A material-system unit can actively dry filament, not merely measure temperature or humidity. |

### Sources

`Sources` contains at most 64 entries. IDs are opaque and should only be used to resolve references in the same topology response.

| Name | Type | Description |
| ---- | :--: | ----------- |
| `Id` | string | Source identifier referenced by `Routes[].SourceId`. |
| `Index` | int | Zero-based normalized order in `Sources`; not a routing or print-selection value. |
| `Name` | string | Human-readable source label, such as `AMS 1 Slot 1` or `External Spool`. |
| `UnitId` | string | ID of the containing unit. Omitted for a standalone source or when its container is unknown. |
| `Position` | int | Platform-local slot position, often within `UnitId`. Positions can repeat across different units. |
| `PrintMappingValue` | int | Platform-native integer used to select this source for a print. It can differ from both `Index` and `Position`. |
| `IsEmpty` | bool | Whether the source currently lacks filament. Omitted when the platform cannot determine it. |
| `Material` | object | Filament or spool currently assigned to the source. Its absence does not imply the source is empty. |
| `PlatformDetails` | object | Optional platform-specific source data that has no portable normalized field. |

### Source Material

| Name | Type | Description |
| ---- | :--: | ----------- |
| `Type` | string | Generic material family, such as `PLA`, `PETG`, `ABS`, or `TPU`. |
| `Name` | string | Platform or spool display name, which can be more specific than `Type`. |
| `ColorHex` | string | Six-digit RGB filament color in `RRGGBB` form without `#`. |
| `RemainingPercent` | float | Estimated filament remaining from `0.0` to `100.0`. Omitted when no reliable estimate exists. |
| `Manufacturer` | string | Filament or spool manufacturer reported by the platform. |

### Tools

`Tools` contains at most 16 physical extrusion endpoints. A tool is not a source; use `Routes` to determine what feeds it.

| Name | Type | Description |
| ---- | :--: | ----------- |
| `Id` | string | Tool identifier referenced by `Routes[].ToolId`. |
| `Index` | int | Zero-based normalized tool number. Use `Id`, not `Index`, for route relationships. |
| `Name` | string | Human-readable tool or tool-head label. |
| `IsActive` | bool | Whether this is the currently selected extrusion tool. Omitted when selection is unknown. |
| `IsPresent` | bool | Whether the physical tool is installed and available. Omitted when presence is unknown. |
| `ActualCelsius` | float | Current measured nozzle or hotend temperature in Celsius. |
| `TargetCelsius` | float | Requested nozzle or hotend temperature in Celsius. Zero normally means the heater is off. |
| `NozzleDiameterMm` | float | Installed nozzle diameter in millimeters for this tool. |
| `PlatformDetails` | object | Optional platform-specific tool data that has no portable normalized field. |

### Units

`Units` contains at most 8 material-system devices or containers. Sources link to their containing unit through `UnitId`.

| Name | Type | Description |
| ---- | :--: | ----------- |
| `Id` | string | Unit identifier referenced by `Sources[].UnitId`. |
| `Index` | int | Zero-based normalized order in `Units`. |
| `Name` | string | Human-readable unit label, such as `AMS 1`. |
| `HumidityPercent` | float | Measured relative humidity from `0.0` to `100.0`, never a vendor-specific condition level. |
| `TemperatureCelsius` | float | Measured internal or enclosure temperature in Celsius. |
| `IsDrying` | bool | Whether the unit is actively drying filament. Omitted when drying state is unknown. |
| `PlatformDetails` | object | Optional platform-specific unit data that has no portable normalized field. |

### Routes

`Routes` contains at most 64 known source-to-tool links. A route can be reported dynamically by the platform or supplied as a fixed relationship for printers whose feeds cannot change.

| Name | Type | Description |
| ---- | :--: | ----------- |
| `SourceId` | string | Must match an `Id` in `Sources`. |
| `ToolId` | string | Must match an `Id` in `Tools`. |
| `State` | string | Optional platform-reported route state, such as `loaded` or `loading`. Omitted when only the relationship is known. |

### Platform Details

`PlatformDetails` can appear on the material system, a source, a tool, or a unit. Its keys vary by `PlatformVersion` and printer platform—for example, Bambu RFID/tray fields or Snapmaker U1 feed and entanglement-sensor data. Treat it as an optional extension object: prefer normalized fields for cross-platform behavior, and only interpret known keys after identifying the platform.


## Feature Flags

`Features` is a bitmask that tells you which [Printer Control APIs](printer-control.md) are supported by the current platform.

| Flag | Value | Control API |
| :--: | :---: | ----------- |
| `FEATURE_LIGHT_CONTROL` | `1` | [Set Light](printer-control.md#set-light) |
| `FEATURE_AXIS_MOVEMENT` | `2` | [Move Axis](printer-control.md#move-axis) |
| `FEATURE_HOMING` | `4` | [Home](printer-control.md#home) |
| `FEATURE_EXTRUSION` | `8` | [Extrude](printer-control.md#extrude) |
| `FEATURE_TEMPERATURE_CONTROL` | `16` | [Set Temp](printer-control.md#set-temp) |
