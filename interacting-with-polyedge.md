---
title: "Interacting with PolyEdge"
nav_order: 3
layout: default
parent: "PolyEdge Documentation"
---

# Interacting with PolyEdge

PolyEdge exposes a running deployment through two interfaces: a web GUI and REST/WebSocket APIs, callable directly.

## The web GUI

`polyedge-interact` serves a web app on port 3000 (production installs run it as the `polyedge-interact` systemd service). Find the URL on your device with:

```bash
polyedge-interact-status
```

This reports whether the service is running and lists every accessible URL (localhost and network IPs).

The GUI provides:
- **Configuration**: set NR parameters (band, frequency, SCS, PRBs, cell ID), gNB/sensor location, and start/stop the radio and ISAC Engine.
- **Monitoring** and **Process Control**: live status and logs for the running processes.
- **Settings**: application-level configuration.

The GUI is a client of its own backend REST API. Everything it does, you can also do programmatically.

<p align="center">
  <img src="assets/images/gui-ss.png" alt="PolyEdge GUI radio configuration screen" width="720" />
</p>
<p align="center"><em>Radio configuration screen: operator, band, frequency, subcarrier spacing, PRBs, cell ID, and gNB position.</em></p>

Radio configuration fields, as shown above:

| Field | Type |
|---|---|
| Operator | text (e.g. "TMO") |
| Band | 3GPP NR band number (e.g. 41, 71, 77); see [Operating PolyEdge](operating-polyedge.md) for enforced band/SCS combinations |
| Frequency | number in Hz (e.g. 3871200000) |
| Subcarrier spacing | dropdown (15/30/60/120/240 kHz) |
| Number of PRBs | number (1-275) |
| Cell ID | number |
| gNB position | latitude, longitude, altitude (meters) |

## The APIs

Everything the GUI does is backed by a REST API on the same port (3000). The detection pipeline exposes its own separate REST/WebSocket/MQTT/TAK outputs. See:

- **[API Reference](api-reference.md)**: full OpenAPI-generated reference for both the orchestrator (:3000) and detection (:8766) APIs.
- **[Operating PolyEdge](operating-polyedge.md)**: walkthrough of starting the radio and setting configuration, with the underlying API calls.
- **[Data Outputs & Integrations](data-outputs-and-integrations.md)**: consuming detections via REST, WebSocket, MQTT, or TAK.

## Authentication

By default, both the orchestrator and detection APIs are unauthenticated on the local network. If your license has an `API_KEY` field embedded, both APIs require a matching `X-API-Key` header. See [Licensing](licensing.md).
