---
title: "API Reference"
nav_order: 9
layout: default
has_children: true
---

# API Reference

PolyEdge exposes two REST APIs plus a real-time WebSocket stream. Pick the one that matches what you're building:

| I want to... | Use | Port |
|---|---|---|
| Start/stop the radio, change NR config, manage presets | [Orchestrator API]({{ site.baseurl }}/api/orchestrator.html) | `:3000` |
| Read detections by polling | [Detection API]({{ site.baseurl }}/api/detection.html) | `:8766` |
| Read detections in real time | Detection WebSocket (`:8765`), same message schema as the Detection API's `/api/v1/latest` | `:8765` |

Both reference pages below are generated directly from the running services' OpenAPI schemas, so they stay in sync with the actual API as it ships.

- **[Orchestrator API reference]({{ site.baseurl }}/api/orchestrator.html)**: radio control, configuration, and process lifecycle (port 3000).
- **[Detection API reference]({{ site.baseurl }}/api/detection.html)**: passive radar detection output (port 8766).

## Authentication

The Orchestrator and Detection APIs share one optional credential: an `API_KEY` field embedded in your PolyEdge license. If no key is embedded, no authentication is required; this is the default. Both reference pages above document the `X-API-Key` header where it applies.

The WebSocket stream has no authentication of its own. Restrict it at the network layer if that matters for your deployment.
