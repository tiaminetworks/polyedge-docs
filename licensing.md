---
title: "Licensing"
nav_order: 7
layout: default
parent: "PolyEdge Documentation"
---

# Licensing

PolyEdge requires a valid, hardware-bound license file (`license.lic`) to run the detection/positioning pipeline. Model weights are never distributed in plaintext — the license carries encrypted access to them.

**Licenses are issued by Tiami Networks, bound to your specific hardware.** You don't generate or manage your own license — contact Tiami Networks for a new deployment, a renewal, or to move to different hardware.

## What's in a license

- **Hardware fingerprint** — bound to the specific machine it was issued for. A license won't validate on different hardware.
- **Expiry date**
- Encrypted access to your trained positioning model
- **Optional API key** — if your license includes one, the orchestrator (:3000) and detection (:8766) REST APIs require a matching `X-API-Key` header on every request. If absent, those APIs are unauthenticated (the default).

## Checking your license

```bash
polyedge-license-status.sh                  # status only
polyedge-license-status.sh --show-api-key   # also reveal the embedded API key, if present
```

Reports valid/invalid, licensee, expiry, and hardware-binding status without modifying anything — safe to run any time.

## What happens if it's invalid or missing

The streamer won't start the detection pipeline at all — license validation happens before initialization. If it expires while running, model-based (ML) predictions are disabled after repeated re-validation failures; the non-ML radar detection pipeline and REST/WebSocket servers continue running.

## Getting a new or updated license

Contact Tiami Networks — for a renewal, a hardware change, or to add/rotate an API key on your license. You'll need your device's hardware identifiers, which `polyedge-license-status.sh` can help confirm are current.
