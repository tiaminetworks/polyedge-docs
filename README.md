# PolyEdge Documentation

PolyEdge is passive ISAC (Integrated Sensing and Communications) software: it uses commercial LTE/NR and wideband RF as illuminators to detect and track targets in real time, with TAK integration for tactical situational awareness, REST/WebSocket/MQTT outputs for programmatic integration, and hardware-bound license-secured model inference.

Full per-topic guides render at **https://tiaminetworks.github.io/polyedge-docs/** ([Installation](installation.md), [Interacting with PolyEdge](interacting-with-polyedge.md), [Updating PolyEdge](updating-polyedge.md), [Operating PolyEdge](operating-polyedge.md), [Data Outputs & Integrations](data-outputs-and-integrations.md), [Licensing](licensing.md), [Model Finetuning](model-finetuning.md), [API Reference](api-reference.md)). This file is the single-document version for anyone reading straight off GitHub.

---

## Table of Contents

1. [Overview](#overview)
2. [The Streamer](#the-streamer)
3. [Installation](#installation)
4. [Interacting with PolyEdge](#interacting-with-polyedge)
5. [Updating PolyEdge](#updating-polyedge)
6. [Configuration Files](#configuration-files)
7. [Starting the System](#starting-the-system)
8. [Data Outputs & Integrations](#data-outputs--integrations)
9. [Licensing](#licensing)
10. [Model Finetuning](#model-finetuning)

---

## Overview

PolyEdge is a real-time ISAC localization system that uses 5G NR downlink synchronization signals (and, more broadly, commercial LTE/NR/wideband RF) as an illuminator to detect and track targets — passive bistatic radar: correlate the direct path from a tower you don't control against the path scattered off a target, without operating your own transmitter.

<p align="center">
  <img src="assets/images/polyedge-isac.gif" alt="PolyEdge passive ISAC bistatic scenario" width="720" />
</p>
<p align="center"><em>PolyEdge ISAC — default passive mode: a cellular illuminator you don't control, a target, and your receive site.</em></p>

<p align="center">
  <img src="assets/images/polyran.png" alt="PolyRAN O-RAN DU with Tiami ISAC" width="600" />
</p>
<p align="center"><em>PolyRAN — the active, gNB-integrated complement: sensing co-located with the cellular stack.</em></p>

## The Streamer

**The streamer is PolyEdge** — the core real-time processing engine that ingests 5G NR signal collection and produces ISAC localization. Everything else in this document (GUI, REST/WebSocket/MQTT/TAK outputs, license validation) is an interface to or a consumer of the streamer, not a separate peer component.

It continuously processes downlink synchronization signal data and produces absolute coordinates via a combination of passive-radar-based processing and ML.

**How it works:**

1. **5G NR frame ingestion** — receives raw downlink 5G NR frames; PolyEdge replicates the complete L1-PHY stack to demodulate and use the downlink telemetry as a sensing resource
2. **Real-time processing** — signal processing and normalization
3. **Model inference** — the trained model predicts absolute target positions
4. **Output generation** — streams results to CSV logs, WebSocket clients, REST API, MQTT, and TAK systems

**Startup sequence:** license validation → configuration loading (`nr_params.json`, `config.json`) → model loading → processing pipeline init → output setup → continuous operation.

**Normal operation:** consistent frame processing (no queue buildup), coordinates at regular intervals, model confidence typically 0.3–0.8, no timeout errors. Queue buildup = processing bottleneck; low confidence = poor RF conditions or model mismatch.

## Installation

PolyEdge ships as four versioned `.deb` packages, installed in order:

| Package | Installs |
|---|---|
| `polyedge-runtime` | Config contract, state, license placeholder |
| `polyedge-radio` | 5G NR UE stack (`nr-uesoftmodem`), `tiami-ringd`, radio libs — independent build chain |
| `polyedge-streamer` | The streamer binary |
| `polyedge-interact` | The GUI web app (port 3000) |

Full detail: [Installation](installation.md).

## Interacting with PolyEdge

The GUI (`polyedge-interact`, port 3000 — find the URL with `polyedge-interact-status`) provides Configuration, Monitoring, and Process Control. Everything it does is backed by a REST API you can also call directly. Full API reference: [API Reference](api-reference.md). Details: [Interacting with PolyEdge](interacting-with-polyedge.md).

<p align="center">
  <img src="assets/images/gui-ss.png" alt="PolyEdge GUI radio configuration screen" width="600" />
</p>
<p align="center"><em>Radio configuration screen — operator, band, frequency, subcarrier spacing, PRBs, cell ID, gNB position.</em></p>

Radio configuration fields, as shown above:

| Field | Type |
|---|---|
| Operator | text (e.g. "TMO") |
| Band | number (e.g. 71, 77, 78, 79) |
| Frequency | number in Hz (e.g. 3871200000) |
| Subcarrier spacing | dropdown (15/30/60/120/240 kHz) |
| Number of PRBs | number (1–275) |
| Cell ID | number |
| gNB position | latitude, longitude, altitude (meters) |

## Updating PolyEdge

Self-update fetches a version manifest and reinstalls the packages — triggered via `POST /api/system/update` from the GUI, gated by `POLYEDGE_ALLOW_UPDATE=true` and a configured `POLYEDGE_UPDATE_MANIFEST_URL`. Details: [Updating PolyEdge](updating-polyedge.md).

## Configuration Files

### `nr_params.json` — 5G NR network parameters

```json
{
    "operator": "OPERATOR_NAME",
    "band": 66,
    "frequency": 2.15268e9,
    "subcarrier_spacing": 15e3,
    "number_of_prbs": 106,
    "gnb_location": {
        "lat": 38.433531,
        "lon": -121.437309,
        "alt": -4.8
    }
}
```

- `operator` — network operator name
- `band` — 5G NR frequency band number (e.g. 66, 71, 78)
- `frequency` — center frequency in Hz
- `subcarrier_spacing` — 15e3 (15 kHz) or 30e3 (30 kHz), band-dependent
- `number_of_prbs` — physical resource block count
- `gnb_location` — **the illuminator's** GPS position (`lat`/`lon`/`alt`) — this is the TX/tower you don't control, not your own sensor

Common bands: **66** (AWS-3, 2.15268e9 Hz, 15kHz SCS, 106 PRBs), **71** (600MHz, 622850000 Hz, 15kHz SCS, 106 PRBs), **78** (3.5GHz, 3.68e9 Hz, 30kHz SCS, 75 PRBs).

### `config.json` — system + TAK configuration

```json
{
  "output_directory": "./tmp/output",
  "csv_file_path": "./tmp/output/tracking_log.csv",
  "license_file": "./license.lic",
  "log_level": "INFO",
  "websocket_port": 8765,
  "model_inference": {
      "enabled": true,
      "device": "cpu",
      "reference_point": {
          "latitude": 38.43575,
          "longitude": -121.43635,
          "altitude": -16.5
      },
      "normalization_type": "zerocenter"
  },
  "processing": {
      "max_queue_size": 100,
      "file_ready_timeout": 5.0,
      "check_interval": 0.1
  },
  "tak": {
      "enabled": false,
      "sensor_heading": 0.0,
      "cot_url": "udp://<host>:6969",
      "client_cert": "/path/to/client-cert.pem",
      "client_key": "/path/to/client-key.pem",
      "ca_file": "/path/to/ca-cert.pem"
  }
}
```

<p align="center">
  <img src="assets/images/ss-pe-config.png" alt="PolyEdge GUI stream/reference-point configuration screen" width="600" />
</p>
<p align="center"><em>PolyEdge configuration screen — sensor reference point.</em></p>

- `model_inference.reference_point` — **your own sensor/receiver's** position. This is distinct from `gnb_location` above (that's the illuminator/TX; this is your RX) — do not confuse the two, but they are not required to be identical either, since bistatic geometry needs both points separately.
- `websocket_port` — real-time streaming (default 8765)
- `tak.cot_url` — scheme decides transport: `tls://host:8089` for a real TAK server (needs `client_cert`/`client_key`/`ca_file`); `udp://host:6969` for certificate-free UDP output (referred to internally as **"FAAD C2"** — same CoT output, no TAK server required)

Full field reference: [Operating PolyEdge](operating-polyedge.md), [Data Outputs & Integrations](data-outputs-and-integrations.md).

## Starting the System

**Current (GUI/`.deb` deployments):** start via the GUI's Configuration page, or directly:

```bash
curl -X POST http://<host>:3000/api/process/init_nrUE -H "Content-Type: application/json" -d '{...nr_params...}'
curl -X POST http://<host>:3000/api/process/stream_main
```

Full walkthrough: [Operating PolyEdge](operating-polyedge.md).

**Common issues:** no GPS output → check reference point alignment; TAK connection failed → verify certificate paths and server connectivity; license errors → check `license.lic` validity and expiry (see [Licensing](licensing.md)).

## Data Outputs & Integrations

Five equivalent output channels — pick whichever fits your integration: **REST** (`:8766`, poll), **WebSocket** (`:8765`, push, same message as REST's `/api/v1/latest`), **MQTT** (AWS IoT Core, per-target messages), **TAK/TLS** (CoT to a real TAK server), **FAAD C2** (CoT over plain UDP, no TAK server or certificates required). Full detail: [Data Outputs & Integrations](data-outputs-and-integrations.md).

## Licensing

A valid, hardware-bound `license.lic` is required to run the detection/positioning pipeline. Licenses are issued by Tiami Networks, bound to your specific hardware — check status locally with `polyedge-license-status.sh`. Full detail: [Licensing](licensing.md).

## Model Finetuning

Secure finetuning pipeline: ground-truth collection (DroneTag Mini) → data collection (`collect_nr_data_launcher.py`) → data setup/processing (`setup_data_launcher.py`) → secure finetuning (`secure_finetune`). Model weights are never exposed in plaintext. Full pipeline: [Model Finetuning](model-finetuning.md).
