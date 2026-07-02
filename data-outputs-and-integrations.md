---
title: "Data Outputs & Integrations"
nav_order: 6
layout: default
parent: "PolyEdge Documentation"
---

# Data Outputs & Integrations

PolyEdge exposes detections through five equivalent channels. None is more "primary" than another; pick whichever fits your integration:

| Channel | Transport | Direction | When to use |
|---|---|---|---|
| REST | HTTP, port 8766 | You poll it | Simple scripts, HTTP-only infrastructure |
| WebSocket | WS, port 8765 | It pushes to you | Real-time, sub-polling-interval latency |
| MQTT | TLS to AWS IoT Core | It pushes to your endpoint | Feeding your own backend, no inbound access needed |
| TAK (TLS) | TLS, port 8089 | It pushes to a listener | Full ATAK/WinTAK/iTAK integration, or a FAAD C2 endpoint configured for TLS |
| FAAD C2 (UDP) | Plain UDP, port 6969 | It pushes to a listener | Direct feed into a Forward Area Air Defense Command and Control endpoint configured for UDP, no certificates required |

REST and WebSocket serve the same message: one aggregate detection per CPI. MQTT, TAK, and FAAD C2 are structurally different from those two and from each other. MQTT sends one flat message per target. TAK and FAAD C2 both send Cursor-on-Target (CoT) XML over the same code path and the same config block; which transport you use (TLS or plain UDP) depends on how the receiving endpoint is configured, not on whether it's a TAK server or FAAD C2.

## REST API

Detection service, port 8766. Poll `GET /api/v1/latest` for the full detection message, or the narrower `/api/v1/{detections,radar,ml,rid,stats}` endpoints for a subset. Full reference: **[API Reference](api-reference.md)**, Detection API.

## WebSocket

Same port range family, 8765. Connect and listen: no request is needed, and there is no authentication on the connection itself (restrict at the network layer if that matters for your deployment). Disable via `config.json`'s `"websocket": {"enabled": false}`.

### Connecting from the command line

[`websocat`](https://github.com/vi/websocat) is a convenient way to check the stream without writing a client:

```bash
sudo wget -qO /usr/local/bin/websocat \
  https://github.com/vi/websocat/releases/latest/download/websocat.x86_64-unknown-linux-musl
sudo chmod a+x /usr/local/bin/websocat
websocat ws://<host>:8765
```

If the ISAC Engine is running and producing detections, messages start printing immediately; no subscribe or handshake step is needed.

### If no messages arrive

Check both ends of the connection:

**On the client:**
- Confirm you can reach the sensor at all: `ping <host>`.
- Confirm there's a route to it: `ip route` (or your platform's equivalent) should show a path to the sensor's subnet.

**On the PolyEdge host:**
- Confirm the client is on a reachable subnet, or that routing between the two is in place.
- Confirm the firewall isn't blocking the port: `sudo ufw status` should not show `8765` (or your configured `websocket.port`) denied.

## MQTT (AWS IoT Core)

Publishes RF detections (Mode 1) and drone/RID detections (Mode 2) as separate, per-target messages to a configured AWS IoT Core topic over TLS.

### Configuration

Set under `mqtt` in `config.json`:

```json
{
  "mqtt": {
    "enabled": true,
    "account_key": "<account_identifier>",
    "broker": "<iot-endpoint>-ats.iot.<region>.amazonaws.com",
    "port": 8883,
    "ca_certs_path": "/path/to/Amazon-root-CA-1.pem",
    "certfile_path": "/path/to/device.pem.crt",
    "keyfile_path": "/path/to/private.pem.key",
    "topic": "<your-topic>"
  }
}
```

All three certificate files (CA root, device certificate, device private key) must be issued and registered in AWS IoT Core, with an IoT Policy granting publish permission on the configured topic.

### Message schema

Every message carries a `schema_version` field so a downstream field addition or removal is traceable. Field additions are always safe for an existing AWS IoT Rule (an unreferenced field is simply not selected); removals or renames require updating the rule's SQL.

**Mode 1 (RF-based detections)**, one message per target:

| Field | Type | Notes |
|---|---|---|
| `schema_version` | string | e.g. `"1.0"` |
| `date_time` | string | ISO 8601 UTC timestamp |
| `device_id` | string | Sensor device identifier |
| `sensor_id`, `tag_id` | string, optional | Sensor and per-target identifiers |
| `mode` | int | `1` for RF-based |
| `target_flag` | int | `1` = drone, `2` = person |
| `longitude`, `latitude`, `altitude` | float | Target GPS coordinates |
| `altitude_type`, `altitude_hae` | string, float, optional | Present only when the source target carries them |
| `confidence` | float | `0.0`-`1.0` |
| `sensor_latitude`, `sensor_longitude`, `sensor_altitude` | float, optional | Sensor (RX) position |
| `gnb_latitude`, `gnb_longitude`, `gnb_altitude` | float, optional | Illuminator (TX) position |
| `sensor_max_radius` | float | Meters, default `1000.0` |
| `cell_id` | optional | Physical cell ID |
| `rcs` | float, optional | Radar cross-section |
| `nr_operator`, `nr_band`, `nr_frequency` | optional | Network/illuminator identity |
| `nr_plmn_id`, `nr_ecid`, `nr_enb_id`, `nr_sector_id` | optional | Populated in NTN mode only |
| `velocity` | float, optional | Target velocity, m/s |
| `rf_meta` | object, optional | RF metadata |
| `sensor_position`, `gnb_position`, `nr_params` | object | Nested duplicates of the flat position/NR fields above, for convenience |
| `radar_detections` | array, optional | Per-detection radar output, see below |

`radar_detections[]` elements carry: `range_m`, `angle_deg`, `magnitude`, `confidence` (core); `bistatic_range_m`, `bistatic_doppler_hz`, `bistatic_range_rate_mps`, `dir_cosines_enu` (bistatic geometry); `target_angle_deg`, `target_aoa_confidence` (per-target angle of arrival); and optionally a nested `direct_path` object with the same angle/confidence/direction-cosine fields for the direct (unscattered) path.

**Mode 2 (drone RID detections)**, one message per detected drone, `mode` always `2` and `target_flag` always `1`. Shares most Mode 1 fields (identifiers, coordinates, confidence, sensor/gNB position, NR params, `rf_meta`, `velocity`), plus RID-specific fields not present in Mode 1:

| Field | Type | Notes |
|---|---|---|
| `manufacturer`, `manufacturer_code` | string, optional | Drone manufacturer identification |
| `operator_location_type_raw`, `operator_location_type` | optional | Operator location type as broadcast by RID |
| `operator_latitude`, `operator_longitude`, `operator_altitude` | float, optional | Pilot/operator position, when broadcast |
| `operator_altitude_unknown` | bool, optional | |
| `operator_location` | optional | Raw operator location payload |
| `drone_meta` | object, optional | `heading`, `vspeed`, `manufacturer`, `basic_id_type`, `operator_location`, `self_id`, `operator_id`, `auth`, and related RID fields |

### AWS IoT Rule example

AWS IoT Core Rules use SQL `SELECT` to extract fields from the incoming message and route them downstream. Nested objects use dot notation, and array elements use bracket notation:

```sql
SELECT
    date_time, device_id, sensor_id, tag_id,
    longitude, latitude, altitude, confidence,
    sensor_position.latitude AS sensor_latitude,
    sensor_position.longitude AS sensor_longitude,
    gnb_position.latitude AS gnb_latitude,
    gnb_position.longitude AS gnb_longitude,
    nr_params.operator AS nr_operator,
    nr_params.band AS nr_band,
    mode, target_flag, velocity,
    radar_detections[0].range_m AS radar_range_m
FROM '<your-topic>'
```

A field must exist in the message payload before a rule can select it; selecting a field that isn't present returns `NULL`, not an error.

## TAK and FAAD C2

Both a real TAK server and **FAAD C2** (Forward Area Air Defense Command and Control) receive detections as Cursor-on-Target (CoT) messages through the same config block and the same code path. The `cot_url` scheme picks the transport, independent of which system is on the receiving end. FAAD C2 accepts either `udp://` or `tls://`, depending on how the FAAD C2 server side is configured; point `cot_url` at whichever transport that endpoint expects.

- **`tls://host:port`** (default port 8089): mutual-TLS. Needs a client certificate, client key, and CA certificate to verify the server. Use this for full ATAK/WinTAK/iTAK integration, or for a FAAD C2 endpoint configured to receive over TLS.

```json
"tak": {
  "enabled": true,
  "sensor_heading": 0.0,
  "cot_url": "tls://<server-ip>:8089",
  "client_cert": "/path/to/client-cert.pem",
  "client_key": "/path/to/client-key.pem",
  "ca_file": "/path/to/ca-cert.pem"
}
```

- **`udp://host:port`** (default port 6969): plain UDP, no certificates required. Use this for a FAAD C2 endpoint configured to receive over UDP.

```json
"tak": {
  "enabled": true,
  "cot_url": "udp://<host>:6969",
  "sensor_heading": 0.0
}
```

Requires `reference_point` (your sensor location, see [Operating PolyEdge](operating-polyedge.md)) to be set before either mode will initialize. `client_cert`, `client_key`, and `ca_file` are ignored entirely in UDP mode.
