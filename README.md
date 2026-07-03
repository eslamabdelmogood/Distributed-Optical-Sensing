# Black Dragon Optical Skin Digital Twin

> A research-grade simulation of distributed fiber-optic structural sensing combined with a three-layer cognitive architecture (BHS), benchmarked against traditional point-sensor industrial monitoring across four fault scenarios.

---

## What This Is

Industrial machines fail. The question is whether you find out in time.

Most factories and infrastructure today use **sparse, fixed-position sensors** — 10–15 temperature probes and vibration gauges spread across a machine surface. This project simulates why that is structurally insufficient, and what a fully distributed sensing architecture with a cognitive decision system can achieve instead.

The **Black Dragon Optical Skin Digital Twin** models a steel machine panel instrumented with a serpentine network of 600 simulated **Fiber Bragg Grating (FBG)** sensors achieving 100% spatial coverage, feeding into a three-layer cognitive system:

- **Bat** — short-horizon forecaster that extrapolates temperature, stress, and damage trends to predict failure before it happens
- **Hermit Crab** — stability evaluator that vetoes actions solving the immediate problem but creating future instability
- **Squid** — meta-controller that dynamically re-weights competing objectives (productivity, safety, energy, structural integrity) as risk evolves

This architecture is compared against a **traditional 15-point-sensor baseline** on identical ground-truth physics, same random seed, same fault injection — so every difference in outcome is attributable to sensing and control, not chance.

---

## Key Results

| Metric | Optical Skin + BHS | Traditional (15 sensors) | Improvement |
|---|---|---|---|
| Detection Time | **5.5s avg** | 137.7s avg* | **~90% faster** |
| Prediction Lead Time | **58.9s** | 0.0s | BHS-only capability |
| Localization Error | **3.2 cells** | 6.0 cells | **~47% more accurate** |
| False Alarms | **0** | 0 | Tie |
| RUL Estimation | **✓ model-based** | ✗ none | BHS-only capability |
| Damage Prevented | **+31.4 pp** vs unmitigated | +0.0 pp | **+31.4 percentage points** |
| Compute Cost | 3.2 ms/step | 0.4 ms/step | Baseline wins (~7× cheaper) |

*\*Baseline detection averaged over Scenario A only — the single scenario where the baseline detected anything at all. In 3 of 4 scenarios, the fault occurred far from any fixed sensor and was never detected within the simulation window.*

### Sensor Density Sweep (15 → 50 → 100 → 600 FBG)

Adding more point sensors helps, but non-monotonically:
- **Scenario C (mechanical overload at panel center):** never detected at 15, 50, or 100 sensors — only the distributed optical skin catches it
- **False alarms increase** with sensor density (0 → 7 → 11 in Scenario B) while the optical skin stays at 0
- **Prediction lead time stays 0** at every point-sensor density — no amount of additional sensors adds a forecasting model

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│              Ground-Truth Physics Panel              │
│   Heat diffusion · Stress · Fatigue/Crack growth    │
└───────────────────────┬─────────────────────────────┘
                        │
          ┌─────────────▼─────────────┐
          │   Layer 1: Optical Skin   │
          │   600 FBG gratings        │
          │   100% spatial coverage   │
          │   IDW field reconstruction│
          └─────────────┬─────────────┘
                        │
          ┌─────────────▼─────────────┐
          │   Layer 2: Reflex Kernel  │
          │   LIF spiking neurons     │
          │   per cell per channel    │
          │   Local threshold reflexes│
          └─────────────┬─────────────┘
                        │
          ┌─────────────▼─────────────┐
          │   Layer 3: BHS Cognition  │
          │  ┌────────┐ ┌──────────┐  │
          │  │  Bat   │ │  Hermit  │  │
          │  │Forecast│ │  Crab    │  │
          │  └────────┘ └──────────┘  │
          │       ┌──────────┐        │
          │       │  Squid   │        │
          │       │Objectives│        │
          │       └──────────┘        │
          └─────────────┬─────────────┘
                        │
          ┌─────────────▼─────────────┐
          │  Layer 4: Actuation       │
          │  Load redistribution      │
          │  Speed reduction          │
          │  Zone isolation           │
          └───────────────────────────┘
```

---

## Fault Scenarios

| | Scenario | Fault Type | BHS Detects | Baseline Detects |
|---|---|---|---|---|
| A | Localized Thermal Fault | Bearing overheating | ✓ 16.2s | ✓ 170.3s (late) |
| B | Crack Initiation & Propagation | Pre-existing flaw + overload | ✓ 0.4s | ✗ never |
| C | Mechanical Overload | 7× load spike at center | ✓ 0.5s | ✗ never |
| D | Combined Heat + Vibration + Stress | Multi-modal failure | ✓ 4.9s | ✗ never |

---

## Deliverables

| File | Description |
|---|---|
| `outputs/dashboard.html` | Self-contained interactive dashboard — field heatmaps, playback, BHS decision timeline, metric charts, verdict |
| `outputs/Black_Dragon_Optical_Skin_Research_Report.docx` | 19-page research report with full methodology, results tables, and embedded charts |
| `outputs/animations/scenario_*.gif` | Animated 6-panel field evolution for each scenario |
| `outputs/metrics_detailed.csv` | Per-scenario per-system raw metric values |
| `outputs/metrics_comparison.csv` | Side-by-side comparison with % improvement per metric |
| `outputs/metrics_aggregate_summary.csv` | Cross-scenario aggregate summary |
| `outputs/sensor_density_sweep.csv` | Sweep across 15 / 50 / 100 point sensors vs 600 FBG |
| `outputs/figures/` | All static comparison charts (PNG) |

---

## Project Structure

```
bdo_skin/
├── sim/
│   ├── physics.py              # Ground-truth FD panel physics
│   ├── optical_skin.py         # Layer 1: Distributed FBG sensing
│   ├── reflex.py               # Layer 2: LIF spiking reflex kernel
│   ├── bhs.py                  # Layer 3: Bat / Hermit Crab / Squid
│   ├── actuation.py            # Layer 4: Physical interventions
│   ├── baseline.py             # Traditional point-sensor comparator
│   ├── scenarios.py            # Fault injection scripts (A–D)
│   ├── orchestrate.py          # Full run orchestration
│   ├── metrics.py              # 8-metric evaluation engine
│   ├── batch_run.py            # Run all scenarios, save results
│   ├── build_metrics.py        # Generate CSVs and charts
│   ├── make_animation.py       # Generate animated GIFs
│   ├── export_dashboard_data.py# Export JSON for dashboard
│   └── sensor_sweep.py         # Sensor density sweep (15/50/100/600)
├── dashboard_template.html     # Dashboard HTML template
├── dashboard_app.js            # Dashboard JavaScript
├── build_dashboard.py          # Assemble final dashboard HTML
├── report_build/               # Report generation (Node.js / docx-js)
│   ├── helpers.js
│   ├── section1–5.js
│   └── build_report.js
└── outputs/                    # All final deliverables
```

---

## Installation & Usage

### Requirements

```bash
Python >= 3.10
numpy scipy matplotlib pandas Pillow playwright
Node.js >= 18 (for report generation only)
```

### Install

```bash
git clone https://github.com/YOUR_USERNAME/black-dragon-optical-skin.git
cd black-dragon-optical-skin
pip install numpy scipy matplotlib pandas Pillow playwright
playwright install chromium   # only needed for dashboard screenshots
```

### Run the full simulation

```bash
# 1. Run all four scenarios (BHS + baseline + unmitigated reference)
python3 sim/batch_run.py

# 2. Generate metrics CSVs and comparison charts
python3 sim/build_metrics.py

# 3. Generate animated GIFs (one per scenario)
python3 sim/make_animation.py

# 4. Export data and build the interactive dashboard
python3 sim/export_dashboard_data.py
python3 build_dashboard.py

# 5. Run the sensor density sweep (15 / 50 / 100 / 600 FBG)
python3 sim/sensor_sweep.py
```

### View results

Open `outputs/dashboard.html` in any modern browser — no server needed, fully self-contained.

### Generate the research report (optional)

```bash
cd report_build
npm install docx
node build_report.js
# Outputs: Black_Dragon_Optical_Skin_Research_Report.docx
```

### Expected runtime

| Step | Time (approx.) |
|---|---|
| Full batch run (4 scenarios × 3 systems) | ~60s |
| Metrics + charts | ~5s |
| Animations (4 GIFs) | ~80s |
| Dashboard build | ~3s |
| Sensor sweep | ~30s |

All tested on a standard CPU (no GPU required).

---

## Physics Model

The ground-truth panel is a 60×40-cell 2D finite-difference mesh (1.2 m × 0.8 m steel panel, dt = 0.05 s) solving four coupled processes:

1. **Heat diffusion** — explicit 5-point Laplacian with convective loss and local fault-driven heat sources
2. **Stress** — mechanical load + constrained thermal expansion stress, concentration factor capped at 3× near damage
3. **Fatigue damage** — Paris-law-inspired accumulation above an endurance limit, with neighbor coalescence modeling micro-crack network growth
4. **Brittle rupture** — cells exceeding damage threshold (0.92) become irreversible hard cracks

The optical skin's FBG readings include gauge-length averaging, sensor noise, and temperature/strain cross-talk — no ground-truth leakage.

---

## Limitations

- 2D simplified geometry, not a validated FEM model — numbers are internally consistent, not calibrated to a specific machine
- Each scenario run once per seed; a rigorous study would sweep seeds for confidence intervals
- The baseline was generously granted a stress-sensing capability most real point-sensor installations don't have
- Compute cost measured on single CPU without architecture-specific optimization

---

## License

MIT License — see [LICENSE](LICENSE)

---

## Citation

If you use this simulation framework in research or derivative work:

```
Black Dragon Optical Skin Digital Twin
Distributed FBG Sensing + BHS Cognitive Architecture Simulation
https://github.com/YOUR_USERNAME/black-dragon-optical-skin
2025
```
