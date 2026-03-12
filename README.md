# InsulinAbaccus
Experimental Type 1 Diabetic Insulin Simulator
# Insulin Action Model

**Version:** 11  
**File:** `insulin-abaccus-v11.html`  
**Stack:** Single-file HTML — React 18, Chart.js 4, Babel standalone (no build step required)

---

## Overview

InsulinAbacus is a self-contained, browser-based insulin dose calculator and monitoring tool that goes well beyond simple carb-ratio arithmetic. It models the real-time physiological factors that affect how insulin behaves in the body — including absorption kinetics, sensitivity modifiers, active insulin on board (IOB), and a 4-hour blood glucose forecast with uncertainty bands.

Because the entire application is a single `.html` file, it requires no installation, no server, and no internet connection after the initial load. All data is persisted to `localStorage` in the browser.

> **Medical Disclaimer:** This tool is intended for informational and educational use only. It is not a substitute for professional medical advice, diagnosis, or treatment. Always consult your diabetes care team before making changes to your insulin regimen.

---

## Features

### Dose Calculator
The dose calculator accepts current blood sugar, carbohydrate intake, and a full set of physiological context inputs. It then computes a recommended bolus dose using the user's insulin-to-carb ratio and correction factor, adjusted in real time by the active modifier stack.

Inputs available on the dose screen include: blood sugar (with optional dual CGM + fingerstick mode), carbohydrate grams, meal macronutrients (fat and protein grams for delayed glucose modelling), meal type (fast carbs, mixed, high fat/protein), fibre class (low/medium/high), heart rate (manually entered or measured via the built-in BPM tap monitor), activity level (Rest through Hard, mapped to MET values), stress level, hydration status, sleep toggle with quality slider, illness toggle, travel/routine disruption toggle, and injection site (abdomen, thigh, arm, or other).

### Physiologic Model

The underlying model is composed of several interacting subsystems:

**Absorption rate (k_abs).** The rate at which injected insulin enters the bloodstream is modulated by BMI, heart rate, physical activity, sleep state and quality, hydration, meal type, and fibre class. Each factor is implemented as a multiplicative coefficient on a baseline absorption rate constant.

**Effective sensitivity (S_eff).** The model computes a dynamic correction factor that reflects how potently one unit of insulin will lower blood sugar at the moment of dosing. The demographic baseline (derived from BMI, age, and sex) is further modulated by current blood sugar level relative to target, exercise state, the dawn phenomenon, recent hypoglycaemia or hyperglycaemia memory, sleep, stress, illness, and travel/circadian disruption.

**Fat and protein delayed glucose (2.1).** Fat and protein grams entered with a meal are converted to a slow-release glucose reservoir that depletes over several hours, contributing to the extended BG impact of high-fat/protein meals.

**Gastric emptying reservoir (2.3).** A two-compartment gastric model tracks the rate at which carbohydrates pass from the gut into the bloodstream, slowed by fat content, protein content, and fibre class.

**Injection site multiplier (1.5).** Each injection site carries a configurable absorption multiplier stored per log entry (abdomen = 1.00, arm = 0.85, other = 0.90, thigh = 0.75 by default).

### Active Insulin on Board (IOB)
The Status tab displays the current insulin on board in units, calculated from all logged doses using exponential decay and plasma clearance kinetics. A breakdown by individual dose is shown alongside a progress bar indicating total remaining active insulin relative to the largest recent dose.

### 4-Hour Blood Glucose Forecast
The forecast projects blood sugar at 30-minute intervals over the next four hours, accounting for active insulin, current sensitivity, fat/protein reservoirs, gastric carbohydrate absorption, and the basal glucose production rate. Each projected point carries a ±uncertainty band; the full-width shaded band on the chart widens under conditions that reduce model confidence. A confidence label (High / Good / Fair / Low / Very Low) is computed from a composite uncertainty scalar that incorporates stress, illness, travel, active meal reservoirs, substance intake, and current IOB.

### Advanced Daily Modifiers
An optional Advanced tab (enabled in Settings) exposes five substance-level inputs with documented physiological rationale:

- **Alcohol** — suppresses hepatic glucose output and slows gastric emptying.
- **Stimulants** (caffeine / prescribed) — reduces insulin sensitivity via adrenergic and adenosine pathways; transiently boosts hepatic glucose production.
- **Depressants** (benzodiazepines / sleep aids) — amplifies the sleep-related sensitivity reduction and slows gastric motility.
- **Nicotine** — impairs peripheral glucose disposal through vasoconstriction.
- **THC / Cannabis** — slows gastric motility via CB1 receptor activation in the enteric nervous system.

### Logs and Charts
The History tab shows a combined Chart.js chart of historical blood sugar readings, insulin doses (purple bars), projected BG trajectory (dashed), and insulin sensitivity percentage (orange line). Below the chart, a full scrollable log table lists every entry with time, BG, dose, carbs, and modifiers. Entries can be individually deleted. Logs can be exported to and imported from CSV, with duplicate-free merging on import.

### Settings
The Settings panel exposes three tiers of configuration. The clinical profile covers carb ratio, correction factor, target BG, insulin duration, low/high thresholds, basal rate, injection site multipliers, BMI, age, sex, and reference values. The simulation parameters panel exposes all physiological coefficients for advanced or clinical users. UI preferences include the dual BG input mode and the Advanced Modifiers tab toggle.

---

## Usage

1. Open `insulin-Abaccus-v11.html` in any modern web browser (Chrome, Safari, Firefox, Edge).
2. Navigate to **Settings** and configure your clinical profile with values from your care team.
3. On the **Dose** tab, enter your current blood sugar and any carbohydrates you are about to eat.
4. Fill in the contextual inputs (activity, stress, sleep, hydration, heart rate, injection site, meal details) for the most accurate modifier stack.
5. Press **Calculate Dose** to receive a recommended bolus, a breakdown of active modifiers, and a 4-hour BG forecast.
6. Press **Log Entry** to save the result. Your log and settings persist automatically between sessions.

---

## Data Persistence and Export

All data is stored in the browser's `localStorage` under versioned keys (`it_profile_v10`, `it_sim_v10`, `it_ui_v10`, `it_logs_v10`). No data is transmitted to any server.

Log entries can be exported as a CSV file from the History tab. The CSV schema includes the following columns:

`timestamp`, `bloodSugar`, `cgm`, `fingerstick`, `carbs`, `insulinDose`, `hr`, `pa`, `sleep`, `sleepQ`, `stress`, `illness`, `hydration`, `travelDay`, `alcoholLevel`, `stimulantLevel`, `depressantLevel`, `nicotineLevel`, `thcLevel`, `site`, `siteMultiplier`, `fat_g`, `protein_g`, `mealType`, `fiberClass`, `kabs_rel`, `seff_mult`, `notes`

Previously exported CSVs can be re-imported and will be merged without duplicates.

---

## Dependencies (loaded from CDN)

| Library | Version | Purpose |
|---|---|---|
| React | 18.2.0 | UI rendering |
| ReactDOM | 18.2.0 | DOM mounting |
| Babel Standalone | 7.23.2 | JSX transpilation in-browser |
| Chart.js | 4.4.1 | BG/insulin history chart |
| Google Fonts | — | DM Sans (UI) + Share Tech Mono (data) |

---

## Model Parameters Reference

All simulation coefficients are documented inline in the source and are adjustable through the Settings panel. Key parameters include:

| Parameter | Default | Description |
|---|---|---|
| `carbRatio` | 12 g/U | Grams of carbohydrate per unit of insulin |
| `corrFactor` | 50 mg/dL/U | Blood sugar drop per correction unit |
| `targetBG` | 100 mg/dL | Dosing target blood sugar |
| `insulinDur` | 4 hr | Base insulin action duration |
| `basalRate` | 0.8 U/hr | Background basal rate |
| `k_abs0` | 0.80 hr⁻¹ | Baseline absorption rate constant |
| `tau_ex` | 3 hr | Exercise effect decay half-life |
| `tau_low` | 4 hr | Post-hypoglycaemia memory half-life |
| `tau_high` | 6 hr | Post-hyperglycaemia memory half-life |
| `u_base` | 18 mg/dL | Base uncertainty band per √hour |

---

## Browser Compatibility

The application requires a browser that supports ES2020+ JavaScript, CSS custom properties, and `localStorage`. It has been designed for both desktop and mobile viewports and is touch-friendly throughout.
