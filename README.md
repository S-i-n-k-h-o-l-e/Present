# Present Experiment

Pilot-ready behavioural self-regulation web app for a 10-second frequency-response task.

## What changed

- Added a structured `trial result` model for each session (raw samples + extracted metrics + classification + risk + feedback + graph data URL).
- Added feature extraction helpers:
  - start/peak/end Hz
  - time to peak
  - average Hz
  - area under curve (trapezoid estimate)
  - variance/stability score
  - largest upward/downward slope
  - rise/fall slope estimate
  - simple plateau detection
- Added a configurable rule-based classifier:
  - `reactive_collapse`
  - `controlled_hold`
  - `driven_sustain`
  - `unclassified`
- Added collapse-risk scoring + risk flag and optional subtle feedback trigger.
- Added local databank (IndexedDB) for saved sessions.
- Added archive UI to reopen trials and overlay selected trials on the graph.
- Added per-trial notes field in results, persisted in local databank records.
- Extended exports:
  - CSV now includes metadata fields (pattern/risk/timestamp/feedback).
  - Added JSON export for complete structured trial payload.
  - Graph PNG export remains available.
- Improved **Send Results**:
  - Uses Web Share API with files when supported.
  - Falls back to `mailto:` with honest manual-attachment instruction (no fake attachment support).

## Classification rules (transparent + tunable)

Rules live in one central config object in `index.html` under `APP_CONFIG.thresholds`.

### 1) `reactive_collapse`
Typical rule signals:
- High peak,
- then sharp drop,
- with steep negative slope.

### 2) `controlled_hold`
Typical rule signals:
- Lower/moderate average output,
- low variance/stable segment,
- plateau detected.

### 3) `driven_sustain`
Typical rule signals:
- Higher average output,
- higher end-state,
- no severe late fall.

If none match → `unclassified`.

## Collapse risk scoring

Simple additive score in range `[0, 1]`, using signals such as:
- steep negative slope,
- sharp post-peak drop,
- low end-Hz trajectory,
- high variance/instability,
- strong negative fall slope estimate.

`collapse_risk_flag` is set when score exceeds `APP_CONFIG.thresholds.collapseRiskFlag`.

## Central tuning values

All main tuning values are centralized in `APP_CONFIG` in `index.html`:
- duration + sample interval
- graph ranges
- pattern thresholds
- risk flag threshold
- feedback defaults
- databank storage names

## Send/share behavior and browser limits

- **Best path:** `navigator.share` + file share (`JSON` + `PNG`) if browser/device supports it.
- **Fallback path:** opens email via `mailto:` containing concise summary and explicit manual attachment instruction.
- Client-side web app cannot reliably auto-attach files to email in unsupported browsers.

## Local databank structure

Uses IndexedDB (`presentExperimentDB`, object store `trials`) where each record stores:
- timestamp + state_before
- detected pattern + risk
- raw samples
- summary metrics
- feedback status
- graph image data URL
- notes field

## Next steps for wearable/physiology integration

- Add ingestion adapters for smartwatch HR/HRV streams.
- Add optional BLE vibration fob driver for richer feedback channels.
- Add multimodal trial schema fields for EEG/EDA/PPG.
- Add per-sensor quality/confidence metrics and synchronization timestamps.
- Add calibration phase per participant to personalize thresholds.
