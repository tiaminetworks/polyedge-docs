---
title: "Data Outputs & Integrations"
nav_order: 6
layout: default
parent: "PolyEdge Documentation"
---

# Data Outputs & Integrations

PolyEdge exposes detections through five equivalent channels — none is more "primary" than another, pick whichever fits your integration:

| Channel | Transport | Direction | When to use |
|---|---|---|---|
| REST | HTTP, port 8766 | You poll it | Simple scripts, HTTP-only infrastructure |
| WebSocket | WS, port 8765 | It pushes to you | Real-time, sub-polling-interval latency |
| MQTT | TLS to AWS IoT Core | It pushes to your endpoint | Feeding your own backend, no inbound access needed |
| TAK (TLS) | TLS, port 8089 | It pushes to a TAK server | Full ATAK/WinTAK/iTAK tactical situational awareness |
| FAAD C2 | Plain UDP, port 6969 | It pushes to a listener | Same CoT output as TAK/TLS, no TAK server or certificates required |

REST and WebSocket serve the **same message** — one aggregate detection per CPI. MQTT, TAK, and FAAD C2 are structurally different from those two and from each other: MQTT sends one flat message per target; TAK and FAAD C2 both send Cursor-on-Target (CoT) XML, differing only in transport (TLS vs. plain UDP) — same underlying output, same code path, different `cot_url` scheme.

## REST API

Detection service, port 8766. Poll `GET /api/v1/latest` for the full detection message, or the narrower `/api/v1/{detections,radar,ml,rid,stats}` endpoints for a subset. Full reference: **[API Reference](api-reference.md)** → Detection API.

## WebSocket

Same port range family, 8765. Connect and listen — no request needed, no authentication on the connection itself (restrict at the network layer if that matters for your deployment). Disable via `config.json`'s `"websocket": {"enabled": false}`.

## MQTT (AWS IoT Core)

Publishes RF detections (Mode 1) and drone/RID detections (Mode 2) as separate, per-target messages to a configured AWS IoT Core topic over TLS. Configure under `mqtt` in `config.json` (broker endpoint, topic, certificate paths). See the main repo's `passive_radar/MQTT/README.md` for the full message schema and AWS IoT Rule setup.

## TAK (TLS) and FAAD C2 (UDP)

Both push detections as Cursor-on-Target (CoT) messages. Same config block, same code path — the `cot_url` scheme picks the transport:

```json
"tak": {
  "enabled": true,
  "sensor_heading": 0.0,
  "cot_url": "tls://<tak-server-ip>:8089",
  "client_cert": "/path/to/client-cert.pem",
  "client_key": "/path/to/client-key.pem",
  "ca_file": "/path/to/ca-cert.pem"
}
```

Requires `reference_point` (your sensor location — see [Operating PolyEdge](operating-polyedge.md)) to be set before either mode will initialize.

- **`tls://host:port`** (default port 8089) — mutual-TLS to a real TAK server. Needs a client certificate, client key, and CA certificate to verify the server. Use this for full ATAK/WinTAK/iTAK integration.
- **`udp://host:port`** (default port 6969) — **FAAD C2**: plain UDP, no certificates required, no TAK server required. Same CoT output as TLS mode.

```json
"tak": {
  "enabled": true,
  "cot_url": "udp://<host>:6969",
  "sensor_heading": 0.0
}
```

`client_cert`/`client_key`/`ca_file` are ignored entirely in FAAD C2 (UDP) mode.
