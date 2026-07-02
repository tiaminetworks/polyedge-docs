---
title: "PolyEdge Documentation"
nav_order: 1
layout: default
---

# PolyEdge Documentation

<!-- ![PolyEdge Logo](assets/images/logo_black.png) -->

Welcome to the PolyEdge documentation. This guide covers installing, operating, and integrating with PolyEdge.

## What is PolyEdge?

PolyEdge is a real-time ISAC (Integrated Sensing and Communications) localization system that uses 5G NR downlink synchronization signals, and more broadly commercial LTE/NR/wideband RF, to track and locate targets, with TAK integration for tactical situational awareness.

The core system is a **passive bistatic radar** pipeline. It correlates a reference path (the direct path from an illuminator you don't control, such as an existing 4G/5G tower) with a surveillance path (energy scattered off targets), forming range-Doppler data, running detection and clutter processing, and, when configured, ML inference and streaming outputs including drone/RID detection.

System components:

- **Real-time, non-cooperative target localization**: converts 5G NR frames to absolute target positions using ISAC.
- **TAK integration**: broadcasts target locations to tactical systems.
- **5G NR interface**: interfaces with 5G networks for signal data collection.
- **License validation**: secure operation with hardware-bound licensing.
- **Process orchestration**: concurrent management of the radio and ISAC Engine processes through the GUI's orchestrator API.

<p align="center">
  <img src="assets/images/polyedge-isac.gif" alt="PolyEdge passive ISAC bistatic scenario" width="720" />
</p>
<p align="center"><em>PolyEdge ISAC: default passive mode. A cellular illuminator you don't control, a target, and your receive site.</em></p>

### PolyRAN: the active, gNB-integrated complement

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
