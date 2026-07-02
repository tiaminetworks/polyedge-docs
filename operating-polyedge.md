---
title: "Operating PolyEdge"
nav_order: 5
layout: default
parent: "PolyEdge Documentation"
---

# Operating PolyEdge

This page covers starting the radio, setting TX/RX locations, and the configuration items that control detection. Full field-by-field reference: [API Reference](api-reference.md), Orchestrator API.

Requires `polyedge-radio` installed (see [Installation](installation.md)) and a valid license (see [Licensing](licensing.md)).

## TX vs. RX location: do not confuse these

PolyEdge is a **passive bistatic** system. It listens to an illuminator you don't control (the TX) and correlates that against what scatters off targets, received at your sensor (the RX). These are two separate, distinct config fields:

| Field | Meaning | Where it lives |
|---|---|---|
| `gnb_location` (`lat`, `lon`, `alt`) | The illuminator's position: the cell tower or gNB you're using as a reference signal source. Not something you control, just something you measure or look up. | NR parameters (`nr_params.json`) |
| `reference_point` (`latitude`, `longitude`, `altitude`) | Your own sensor/receiver's position. | Stream configuration (`config.json`'s `model_inference.reference_point`) |

Get this wrong and range/bearing calculations will be wrong even if signal processing is working perfectly. These two points define the bistatic geometry.

## NR parameters

Set via the GUI's Configuration page, or `POST /api/config/nr_params`:

| Field | Meaning |
|---|---|
| `operator` | Network operator label (e.g. `"TMO"`, `"ATT"`, `"VZW"`) |
| `band` | 3GPP NR band number (e.g. 71, 78) |
| `frequency` | DL carrier center frequency, Hz |
| `subcarrier_spacing` | SCS in Hz, band-dependent, see the validation table below |
| `number_of_prbs` | Physical resource block count (1-275) |
| `cell_id` | Physical cell ID, or a list of candidate PCIs to scan |

### Band / SCS validation

These bands have an enforced SCS whitelist; an unsupported SCS for one of them is rejected at radio-start time:

| Band | Allowed SCS (Hz) |
|---|---|
| 71 | 15000 |
| 78, 79 | 15000, 30000 |
| 257, 258, 260 | 15000, 30000, 60000 |

Other bands (e.g. 41, 77) are accepted without an SCS-specific check.

## Starting the radio

From the GUI's Configuration page, or directly against the orchestrator API. Both `init_nrUE` (the radio) and `stream_main` (the ISAC Engine) are orchestrator-managed processes with the same start/stop/status contract, so the request/response shapes below apply to either one.

### 1. Initialize the radio (`init_nrUE`)

`POST /api/process/init_nrUE`, `Content-Type: application/json`, body is the NR parameter set:

```bash
curl -X POST http://<host>:3000/api/process/init_nrUE \
  -H "Content-Type: application/json" \
  -d '{
    "operator": "TMO", "band": 71, "frequency": 622850000,
    "subcarrier_spacing": 15000, "number_of_prbs": 106, "cell_id": 842,
    "gnb_location": {"lat": 38.431722, "lon": -121.438829, "alt": 25.0}
  }'
```

`band`, `frequency`, `subcarrier_spacing`, and `number_of_prbs` are required; `operator`, `cell_id`, and `gnb_location` are optional. On success:

```json
{
  "success": true,
  "pid": 12345,
  "message": "Process init_nrUE started successfully"
}
```

A missing required field returns `400`:

```json
{
  "error": "Missing required parameters: band, frequency, subcarrier_spacing, number_of_prbs"
}
```

### 2. Start the ISAC Engine (`stream_main`)

Once `init_nrUE` is running, start the ISAC Engine. No body is required:

```bash
curl -X POST http://<host>:3000/api/process/stream_main
```

```json
{
  "success": true,
  "pid": 12346,
  "message": "Process stream_main started successfully"
}
```

If the ISAC Engine binary isn't where the orchestrator expects it (a broken or missing symlink from installation), this returns `500` with an error identifying the missing path. Re-running `install-polyedge-interact-service.sh` (see [Installation](installation.md)) resolves this in the normal case.

### Stopping, checking status, and listing processes

Both processes share the same lifecycle endpoints, keyed by `processId` (`init_nrUE` or `stream_main`):

| Action | Method | Endpoint |
|---|---|---|
| Stop a process | `POST` | `/api/process/{processId}/stop` |
| Get one process's status | `GET` | `/api/process/{processId}/status` |
| List all processes | `GET` | `/api/processes` |

```bash
curl -X POST http://<host>:3000/api/process/stream_main/stop
curl http://<host>:3000/api/process/init_nrUE/status
curl http://<host>:3000/api/processes
```

Poll `GET /api/processes` after starting `init_nrUE` and wait for its status to report running before starting `stream_main`; starting the ISAC Engine before the radio is up will fail once it tries to read live signal data. Stop in reverse order: `stream_main` first, then `init_nrUE`.

## Stream / detection configuration

`config.json`'s `model_inference` block controls the detection/positioning pipeline:

<p align="center">
  <img src="assets/images/ss-pe-config.png" alt="PolyEdge GUI stream/reference-point configuration screen" width="720" />
</p>
<p align="center"><em>PolyEdge configuration screen: sensor reference point. This is the RX position, distinct from gNB position above.</em></p>

| Field | Meaning |
|---|---|
| `enable` | Turn model-based positioning on/off |
| `device` | `cpu` or `cuda` |
| `reference_point` | Your sensor's lat/lon/alt (see TX vs RX above) |
| `use_case` | e.g. `"gps"`, selects which trained model/normalization to use |

`websocket.port` (default 8765) and the `tak` and MQTT blocks are covered in [Data Outputs & Integrations](data-outputs-and-integrations.md).

## Presets

Named configuration bundles can be applied without editing files directly. Use `GET /api/config/presets` to list them and `POST /api/config/select_presets` to apply one. See the [API Reference](api-reference.md) for the full presets contract.
