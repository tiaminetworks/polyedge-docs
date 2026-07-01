---
title: "PolyEdge Documentation"
nav_order: 1
layout: default
---

# PolyEdge Documentation

<!-- ![PolyEdge Logo](assets/images/logo_black.png) -->

Welcome to the PolyEdge documentation. This guide covers installing, operating, and integrating with PolyEdge.

## What is PolyEdge?

PolyEdge is **passive ISAC (Integrated Sensing and Communications) software** from Tiami Networks: it uses commercial LTE/NR and wideband RF as illuminators to detect and track targets, runs streaming inference, and ships results to operational tooling — without operating your own radar transmitter.

The core system is a **passive bistatic radar** pipeline: it correlates a reference path (the direct path from an illuminator you don't control, such as an existing 4G/5G tower) with a surveillance path (energy scattered off targets), forming range–Doppler data, running detection and clutter processing, and — when configured — ML inference and streaming outputs, including drone/RID detection.

<p align="center">
  <img src="assets/images/polyedge-isac.gif" alt="PolyEdge passive ISAC bistatic scenario" width="720" />
</p>
<p align="center"><em>PolyEdge ISAC — default passive mode: a cellular illuminator you don't control, a target, and your receive site.</em></p>

### PolyRAN — the active, gNB-integrated complement

When sensing is co-located with the cellular stack on the gNB side, the network can schedule uplink-centric resources (e.g. SRS or PUSCH-framed uplink) so the DU runs Tiami ISAC alongside 5G-PHY.

<p align="center">
  <img src="assets/images/polyran.png" alt="PolyRAN O-RAN DU with Tiami ISAC" width="720" />
</p>

---

## Where to start

| Need | Read |
|---|---|
| Install PolyEdge on a device | [Installation](installation.md) |
| Use the web GUI or call the APIs | [Interacting with PolyEdge](interacting-with-polyedge.md) |
| Push a new software version to a deployed device | [Updating PolyEdge](updating-polyedge.md) |
| Start the radio, set TX/RX locations, configure the carrier | [Operating PolyEdge](operating-polyedge.md) |
| Consume detections via REST, WebSocket, MQTT, or TAK | [Data Outputs & Integrations](data-outputs-and-integrations.md) |
| Check your license status | [Licensing](licensing.md) |
| Collect ground truth and finetune the positioning model | [Model Finetuning](model-finetuning.md) |
| Full API reference (OpenAPI-generated) | [API Reference](api-reference.md) |
