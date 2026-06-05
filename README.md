# NirogGyan Corporate Wellness Dashboard

A three-page interactive corporate health intelligence platform built for HR teams and executives. Transforms raw employee lab data into actionable wellness insights across three distinct analytical lenses.

**Live Site:** https://aksharreddy.github.io/niroggyan-corporate-wellness/

---

## Pages

### Dashboard (`index.html`)
Executive-level org health overview. Answers: *how healthy is the workforce overall?*

- **4 KPI cards** — Health Index (78/100), Participation rate (82%), Largest Risk (Metabolic 40%), Priority Action (Lipid 28%)
- **Department Health Ranking** — composite wellness score per team with org benchmark line
- **AI Workforce Summary** — auto-generated narrative surfacing department, age, and location risk patterns
- **Aggregated Diagnostic Profiles** — 8 clinical parameter rows with % out-of-range, status (At Risk / Monitor / Healthy), progress bars, and expandable AI intervention recommendations
- **Workforce Risk Profiling** — three-tier cards (Low / Monitor / High) broken down by department, age group, and location

### AI Strategic Insights (`ai_insights.html`)
Clinical intelligence layer. Answers: *what is the underlying health state of the org as a system?*

- **Org Vital Signs** — 3 clinical monitor cards (Cardiovascular 34% Warning · Metabolic 40% Critical · Nutritional 52% Elevated) with pulsing status indicators and system-level waveforms
- **Clinical Tier Profiles** — average biomarker values (VitD, LDL, HbA1c, B12, Hemoglobin) per risk tier, showing the clinical fingerprint that separates Low / Monitor / High cohorts. Monitor card includes escalation probability (28%)
- **Deep Dive Health Intelligence** — tabbed deep-dive (Vitamin D · Lipid Profile · Vitamin B12) with donut chart, diet & lifestyle tips, clinical complications, AI health insight, and strategic HR interventions
- **Intervention Simulator** — toggle-based scenario modeller showing projected health score improvement
- **AI-Discovered Risk Clusters** — ML-identified cohort patterns that don't surface in standard demographic filters

### Workforce Risk Profiling (`workforce.html`)
Cohort analytics layer. Answers: *who specifically is at risk and where?*

- **KPI Overview** — Low Risk 62% (278 emp) · Monitoring 26% (117 emp) · High Risk 12% (54 emp) · Avg Score 78
- **Risk Score Distribution** — health score density curve across the full workforce
- **Priority Interventions** — department-specific alert cards with recommended actions
- **Productivity Risk Map** — bubble chart: headcount × % high-risk per department
- **AI-Generated Intervention Personas** — 3 behavioural health clusters (Sedentary Operator · Depleted Developer · Stressed Achiever) with anomaly bars and intervention strategies
- **Parameter Risk Intelligence** — 3-tab panel:
  - *By Dept* — each clinical parameter broken down across all 5 departments
  - *By Gender* — Female vs Male split per parameter with pp-gap callouts
  - *Co-occurrence* — 5×5 purple heat map showing which parameters flag together
- **Intervention Prioritization Matrix** — 2×2 impact vs effort quadrant
- **High-Risk Cohort Explorer** — searchable table with primary risk driver and priority per cohort

---

## Tech Stack

| Layer | Technology |
|---|---|
| Markup | HTML5 |
| Styling | Tailwind CSS (CDN, v3 with forms + container-queries plugins) |
| Typography | Inter (Google Fonts) |
| Icons | Material Symbols Outlined (Google Fonts) |
| Charts | Pure SVG — gauges, donuts, distribution curves, bubble chart |
| Animations | CSS keyframes + IntersectionObserver-driven entrance animations |
| Interactivity | Vanilla JavaScript — no frameworks |
| Deployment | GitHub Pages (static, no build step) |

---

## Data Model

The dashboard is designed to connect to a CSV with **68 employee fields**. See `DATA_AND_LOGIC.md` for the complete field catalogue, reference ranges, and panel logic.

**Two fields need to be added to the existing CSV:**

| Field | Type | Description |
|---|---|---|
| `Location` | String | Office city (Surat / Hyderabad / Pune / etc.) |
| `HbA1c` | Numeric | Glycated haemoglobin % |

**Key computation constants:**

| Constant | Value |
|---|---|
| Total headcount | 548 |
| Tested employees | 449 (82%) |
| Health score formula | Weighted composite across 8 clinical panels |

For full implementation logic — how every number on every page is computed from raw CSV fields — see `IMPLEMENTATION_SPEC.md`.

---

## Clinical Panels & Weights

The composite health score weights 8 diagnostic panels:

| Panel | Weight | Primary Flag Fields |
|---|---|---|
| Lipid Profile | 20% | Total Cholesterol, Triglycerides, LDL, HDL |
| CBC / Blood Counts | 15% | Hemoglobin (gender-adjusted), Platelets, WBC |
| Vitamin D | 15% | Vitamin D (< 20 ng/mL = At Risk) |
| HbA1c / Diabetes | 15% | HbA1c, Fasting Blood Sugar |
| Vitamin B12 | 10% | VITAMIN B12 (< 200 = At Risk, 200–299 = Monitor) |
| Kidney Function | 10% | Creatinine, BUN, Uric Acid (gender-adjusted) |
| Thyroid Profile | 10% | TSH (outside 0.35–5.6 = flag) |
| Liver Function | 5% | SGPT (ALT), SGOT (AST) |

---

## Features

- **Dark mode** — full dark/light toggle, persisted in localStorage, keyboard shortcut `Cmd/Ctrl+D`
- **Guided Walkthrough** — step-by-step product tour on the Dashboard; clicking Walkthrough on any page redirects to the Dashboard and auto-starts the tour
- **Filters** — Department / Gender / Age Group / Location filters on the Dashboard drive all clinical profile calculations
- **Interactive tabs** — Parameter Risk Intelligence (By Dept / By Gender / Co-occurrence) with animated bar re-renders on tab switch
- **Expandable rows** — Diagnostic Profiles accordion with AI recommendation text per parameter
- **Responsive** — mobile-first, collapses gracefully across breakpoints

---

## Deployment

This is a static site. No build step required.

```bash
# Clone
git clone https://github.com/aksharReddy/niroggyan-corporate-wellness.git

# Open locally
open index.html

# Or serve with any static server
npx serve .
```

GitHub Pages serves directly from the `main` branch root. Any `git push` to `main` deploys within ~60 seconds.

---

## Documentation

| File | Contents |
|---|---|
| `IMPLEMENTATION_SPEC.md` | Every metric on every page — field requirements, computation logic, formula-level detail |
| `DATA_AND_LOGIC.md` | Complete 68-field catalogue, 5 sample employee records with all fields populated, full worked computation on sample data |

---

## Organisation

**NirogGyan** · Corporate Health Intelligence · Q2 2024  
Data integrity protected with enterprise-grade encryption.
