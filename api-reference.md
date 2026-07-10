---
title: "API Reference"
nav_order: 9
layout: default
has_children: true
---

# API Reference

## On-device PolyEdge APIs

PolyEdge exposes two REST APIs plus a real-time WebSocket stream on the sensor device:

| I want to... | Use | Port |
|---|---|---|
| Start/stop the radio, change NR config, manage presets | [Orchestrator API]({{ site.baseurl }}/api/orchestrator.html) | `:3000` |
| Read detections by polling | [Detection API]({{ site.baseurl }}/api/detection.html) | `:8766` |
| Read detections in real time | Detection WebSocket (`:8765`), same message schema as the Detection API's `/api/v1/latest` | `:8765` |

Both on-device reference pages are generated from OpenAPI schemas:

- **[Orchestrator API reference]({{ site.baseurl }}/api/orchestrator.html)** — radio control, configuration, and process lifecycle (port 3000).
- **[Detection API reference]({{ site.baseurl }}/api/detection.html)** — passive radar detection output (port 8766).

### Authentication (on-device)

The Orchestrator and Detection APIs share one optional credential: an `API_KEY` field embedded in your PolyEdge license. If no key is embedded, no authentication is required; this is the default. Both reference pages above document the `X-API-Key` header where it applies.

The WebSocket stream has no authentication of its own. Restrict it at the network layer if that matters for your deployment.

---

## Cloud application APIs

| I want to... | User guide | API reference |
|---|---|---|
| Use the Asset Tracker dashboard | [Asset Tracker Dashboard]({{ '/asset-tracker-dashboard.html' | relative_url }}) | [Asset Tracker Dashboard API]({{ site.baseurl }}/asset-tracker-dashboard/dashboard.html) |
| Use the RF Planner | [RF Planner]({{ '/rf-planner.html' | relative_url }}) | [RF Planner API]({{ site.baseurl }}/rf-planner/rf-planner.html) |
