---
title: "Licensing"
nav_order: 7
layout: default
parent: "PolyEdge Documentation"
---

# Licensing

PolyEdge requires a valid, hardware-bound license file (`license.lic`) to run the detection/positioning pipeline. Licenses are issued by Tiami Networks and bound to your specific hardware. Contact Tiami Networks for a new deployment, a renewal, or a hardware change.

## Checking your license

```bash
polyedge-license-status.sh                  # status only
polyedge-license-status.sh --show-api-key   # also reveal the embedded API key, if present
```

This reports valid/invalid status, licensee, and expiry. It does not modify anything, so it can be run at any time.

## If the ISAC Engine won't start

A missing, expired, or hardware-mismatched license is one of several reasons `stream_main` can fail to start. Confirm the cause before contacting support:

1. Run `polyedge-license-status.sh`. If it reports invalid, the license is the cause.
2. If it reports valid and the ISAC Engine still won't start, check `journalctl -u polyedge-interact -f` for the actual startup error. The cause is something other than licensing.

If the license is confirmed invalid, contact Tiami Networks with the output of `polyedge-license-status.sh` to arrange a renewal or replacement.

## API key

If your license includes an API key, the orchestrator (:3000) and detection (:8766) REST APIs require a matching `X-API-Key` header on every request. Retrieve it with `polyedge-license-status.sh --show-api-key`.
