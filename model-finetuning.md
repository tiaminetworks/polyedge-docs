---
title: "Model Finetuning"
nav_order: 8
layout: default
parent: "PolyEdge Documentation"
---

# Model Finetuning

PolyEdge supports secure finetuning of the positioning model to adapt to a new environment. Finetuning requires a valid license and never exposes model weights in plaintext — encryption and access controls apply throughout.

## Pipeline overview

1. **Ground truth collection** — a DroneTag Mini provides sub-meter-accuracy position logging as training ground truth
2. **Data collection** — collect NR/5G signal data synchronized with the ground-truth positions
3. **Data setup & processing** — clean, normalize, and validate the collected data
4. **Secure finetuning** — train against the processed data with weights protected by your license

## 1. Ground truth — DroneTag Mini

A precision positioning device ([dronetag.cz](https://dronetag.cz/en/products/dronetag-mini/)) mounted on the platform used for data collection. Configure it to 4 Hz update rate and JSON position export before starting (Configuration → Direct/Broadcast Remote ID in the DroneTag mobile app). Start DroneTag Mini logging *before* starting PolyEdge collection, and stop it *after* PolyEdge collection finishes.

## 2. Data collection

Requires: DroneTag Mini logging, a valid `license.lic`, and NR signal hardware/source.

```bash
python3 organizer/collect_nr_data_launcher.py --duration 300 --samples 5
```

`--duration` (seconds, default 1200) and `--samples` (default 1) control collection length and repeat count.

## 3. Data setup & processing

```bash
python3 setup_data_launcher.py \
  --gps-file /path/to/position_export.json \
  --reference-point "37.7749,-122.4194,10.0"
```

`--reference-point` must be the same sensor reference point used during collection (`lat,lon,alt`). This step cleans/normalizes the DroneTag Mini export, verifies RF/position synchronization, and prepares the processed dataset for training.

## 4. Secure finetuning

```bash
./secure_finetune \
  --reference-point "37.7749,-122.4194,10.0" \
  --nr-params "nr_params.json" \
  --epochs 100 \
  --lr 1e-5 \
  --batch-size-train 32 \
  --report-enhanced-metrics \
  --accuracy-profile
```

No explicit data path is needed — `secure_finetune` auto-detects the processed output from step 3. Key arguments:

| Argument | Meaning |
|---|---|
| `--reference-point` (required) | Must match the setup step's reference point |
| `--nr-params` (required) | Path to your `nr_params.json` |
| `--epochs` | Finetuning epochs (default 50) |
| `--lr` | Learning rate (default 1e-5 — lower than initial training) |
| `--license-file` | Defaults to `license.lic` |
| `--sanitize` | Drop invalid samples from the dataset |

The resulting encrypted model integrates directly with the streaming system — no manual deployment step.

---

*This page is reconciled from an earlier internal user guide and hasn't been independently re-verified against the current script paths/flags this session — spot-check `collect_nr_data_launcher.py`, `setup_data_launcher.py`, and `secure_finetune` against your installed version before relying on exact flag names.*
