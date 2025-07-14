# Quick Introduction and Small Disclaimer

I really hope you find this guide helpful. I am no authority on this protocol and you should treat this all with a grain of salt, and assume that any of this could change at any time. I did my best to document what I found.

At the time of writing (07-13-2025) my Elegoo Centauri Carbon is on Firmware version 1.1.29

## What is SDCP
SDCP, specifically SDCP 3.0, from what I could tell is an MQTT over WebSockets protocol developed by cbd-tech; a Chinese manufacturer of 3D printer mainboards as well as the company behind ChituBox, the popular slicing software for resin printers.

Elegoo appears to have partnered with cbd-tech on most likely the mainboard, and has thus landed on using SDCP for their latest and greatest...the Elegoo Centauri Carbon.

## SDCP implementation varies

Just because a printer uses SDCP doesn't mean it will use the same cmd codes, status codes, or payloads. This is a per-printer specification. So Cmd code 129 might mean pause the print on the Elegoo Centauri Carbon but might mean start on some other printer.

## Connecting To your Printer

The Centauri Carbon's websocket server is available at your printers local IP `ws://192.168.XX.XX/websocket`, it requires no handshake or authentication so you can imediatly start sending events via something like postman.

## Keeping the Connection open

By default the websocket connection will get closed after 60 seconds of nothing from the client. From my experience any cmd will work to reset but you can also send a simple "ping" message body.

## Known Commands and what they do...

| **Cmd** | **Function**          | **Description**                                                                                        | **Example**                             |
| ------: | --------------------- | ----------------------------------------------------------------------------------------------------- | --------------------------------------- |
|     `0` | Request Status Update | Prompts the printer to send its current status immediately.                                           | [View](#cmd-0--request-status-update)   |
|     `1` | Get Device Attributes | Returns printer name, firmware, capabilities, IP, MAC, supported features, and other device metadata. | [View](#cmd-1--get-device-attributes)   |
|   `128` | Start Print Job       | Begins printing a selected G-code file with optional start layer and toggles.                         | [View](#cmd-128--start-print-job)       |
|   `129` | Pause Print           | Pauses the active print job.                                                                          | [View](#cmd-129--pause-print)           |
|   `130` | Cancel Print          | Cancels the current print and returns the printer to an idle state.                                   | [View](#cmd-130--cancel-print)          |
|   `131` | Resume Print          | Resumes a paused print.                                                                               | [View](#cmd-131--resume-print)          |
|   `134` | Unknown / No-op       | Returns `Ack`, but no side effects observed.                                                          | [View](#cmd-134--unknown--no-op)        |
|   `258` | List Files            | Retrieves G-code files from a specified directory, e.g., `/local`.                                    | [View](#cmd-258--list-files)            |
|   `320` | Get Print History     | Returns UUIDs for historical prints.                                                                  | [View](#cmd-320--get-print-history)     |
|   `386` | Get Camera Stream URL | Enables the printer's camera and returns MJPEG video URL.                                             | [View](#cmd-386--get-camera-stream-url) |
|   `403` | Toggle Light          | Turns second light on/off. RGB appears unsupported.                                                   | [View](#cmd-403--toggle-light)          |

---

## Status Message Example

```json
{
  "Status": {
    "CurrentStatus": [0],
    "TimeLapseStatus": 0,
    "PlatFormType": 0,
    "TempOfHotbed": 67.49338678423711,
    "TempOfNozzle": 115.34388355923741,
    "TempOfBox": 26.42958339525779,
    "TempTargetHotbed": 0,
    "TempTargetNozzle": 0,
    "TempTargetBox": 0,
    "CurrenCoord": "202.00,264.50,24.59",
    "CurrentFanSpeed": {
      "ModelFan": 0,
      "AuxiliaryFan": 0,
      "BoxFan": 0
    },
    "ZOffset": 0.00000000000001,
    "LightStatus": {
      "SecondLight": 1,
      "RgbLight": [0, 0, 0]
    },
    "PrintInfo": {
      "Status": 8,
      "CurrentLayer": 0,
      "TotalLayer": 165,
      "CurrentTicks": 0,
      "TotalTicks": 9749,
      "Filename": "",
      "TaskId": "",
      "PrintSpeedPct": 100,
      "Progress": 0
    }
  },
  "MainboardID": "608715130105041800009c0000000000",
  "TimeStamp": 1752339395,
  "Topic": "sdcp/status/608715130105041800009c0000000000"
}
```
### Status codes I've observed in PrintInfo.Status
| Code | Meaning                | Description                                                                            |
| ---- | ---------------------- | -------------------------------------------------------------------------------------- |
| `0`  | **Idle**               | No print in progress. Default state after power-on or when print is canceled/finished. |
| `8`  | **Preparing to Print** | Print job accepted, beginning warm-up or calibration routines.                         |
| `9`  | **Starting Print**     | Print has begun — may be homing, priming, or performing initial G-code steps.          |
| `10` | **Paused**             | Print is paused by user or error.                                                      |
| `13` | **Printing (Active)**  | Actively printing layer-by-layer; heaters, motion, and timers engaged.                 |


## ACK Message Example

```json
{
  "Id": "979d4C788A4a78bC777A870F1A02867A",
  "Data": {
    "Cmd": 128,
    "Data": { "Ack": 2 },
    "RequestID": "3333",
    "MainboardID": "608715130105041800009c0000000000",
    "TimeStamp": 1752339218
  },
  "Topic": "sdcp/response/608715130105041800009c0000000000"
}
```

| Ack Code | Meaning            | Description                                                              |
| -------- | ------------------ | ------------------------------------------------------------------------ |
| `0`      | **Success**        | Command was received and executed successfully.                          |
| `1`      | **Failure/Error**  | Command failed (e.g., invalid operation or unsupported parameter).       |
| `2`      | **File Not Found** | Saw when attempting to start a print job with a non-existent file. |


## 🔍 Examples

### Cmd: 0 — Request Status Update

```json
{
  "Id": "",
  "Data": {
    "Cmd": 0,
    "Data": {},
    "RequestID": "abc123",
    "MainboardID": "",
    "TimeStamp": 1752333800479,
    "From": 1
  }
}
```

### Cmd: 1 — Get Device Attributes

```json
{
  "Id": "",
  "Data": {
    "Cmd": 1,
    "Data": {},
    "RequestID": "e4518e17dc004ae3b1cd88fc01ec3c72",
    "MainboardID": "",
    "TimeStamp": 1752333800479,
    "From": 1
  }
}
```

### Cmd: 128 — Start Print Job

```json
{
  "Id": "",
  "Data": {
    "Cmd": 128,
    "Data": {
      "Filename": "/local/some_file.gcode",
      "StartLayer": 0,
      "Calibration_switch": 0,
      "PrintPlatformType": 0,
      "Tlp_Switch": 0
    },
    "RequestID": "bc6990ec21c94b9694f7e0be31b99514",
    "MainboardID": "",
    "TimeStamp": 1752338967263,
    "From": 1
  }
}
```

### Cmd: 129 — Pause Print

```json
{
  "Id": "",
  "Data": {
    "Cmd": 129,
    "Data": {},
    "RequestID": "25d142c593774914aeaa658f5e9397b1",
    "MainboardID": "",
    "TimeStamp": 1752338252077,
    "From": 1
  }
}
```

### Cmd: 130 — Cancel Print

```json
{
  "Id": "",
  "Data": {
    "Cmd": 130,
    "Data": {},
    "RequestID": "490f5d7547294bb2a35ebbce2de1e5df",
    "MainboardID": "",
    "TimeStamp": 1752338652034,
    "From": 1
  }
}
```

### Cmd: 131 — Resume Print

```json
{
  "Id": "",
  "Data": {
    "Cmd": 131,
    "Data": {},
    "RequestID": "b5a7c589bb6842eca994a66c02ebac8a",
    "MainboardID": "",
    "TimeStamp": 1752338355217,
    "From": 1
  }
}
```

### Cmd: 134 — Unknown / No-op

```json
{
  "Id": "",
  "Data": {
    "Cmd": 134,
    "Data": {},
    "RequestID": "420451207d6e44ec9131bdba6afb5991",
    "MainboardID": "",
    "TimeStamp": 1752335529,
    "From": 1
  }
}
```

### Cmd: 258 — List Files

```json
{
  "Id": "",
  "Data": {
    "Cmd": 258,
    "Data": {
      "Url": "/local"
    },
    "RequestID": "83fe754d9e654b51b7de04a68fa6258f",
    "MainboardID": "",
    "TimeStamp": 1752335626,
    "From": 1
  }
}
```

### Cmd: 320 — Get Print History

```json
{
  "Id": "",
  "Data": {
    "Cmd": 320,
    "Data": {},
    "RequestID": "d6a3fbc59ddd43c089156d274ddbbd81",
    "MainboardID": "",
    "TimeStamp": 1752335377,
    "From": 1
  }
}
```

### Cmd: 386 — Get Camera Stream URL

```json
{
  "Id": "",
  "Data": {
    "Cmd": 386,
    "Data": {
      "Enable": 1
    },
    "RequestID": "d40d5619eb2441f1ac1881b70c47d768",
    "MainboardID": "",
    "TimeStamp": 1752335729,
    "From": 1
  }
}
```

### Cmd: 403 — Toggle Light

```json
{
  "Id": "",
  "Data": {
    "Cmd": 403,
    "Data": {
      "LightStatus": {
        "SecondLight": true,
        "RgbLight": [0, 0, 0]
      }
    },
    "RequestID": "78bb6259e2f94a4ea9c13beee3a5b109",
    "MainboardID": "",
    "TimeStamp": 1752337353254,
    "From": 1
  }
}
```

# Sources and Acknowledgements

- [cassini CLI tool](https://github.com/vvuk/cassini) — Python CLI lib by Vladimir Vukicevic, appears to be no longer supported.
- [SDCP-Smart-Device-Control-Protocol-V3.0.0](https://github.com/cbd-tech/SDCP-Smart-Device-Control-Protocol-V3.0.0) — SDCP spec from CBD Tech
- [sdcpapi By Bushvin](https://gitlab.com/bushvin/sdcpapi) — Most complete example of a lib for interacting with SDCP, specifically good resource for the Elegoo flavor.