# NirogGyan Corporate Wellness — Implementation Specification

Complete field-by-field, number-by-number breakdown of every metric on every page, with the exact computation logic required to replace hardcoded values with live CSV data.

---

## Global Constants

| Constant | Value | Source |
|---|---|---|
| `TOTAL_HEADCOUNT` | 548 | Stored externally — total employees invited for testing |
| `TOTAL_TESTED` | 449 | COUNT of rows in CSV with a valid test result |
| `TOTAL_NOT_TESTED` | 99 | `TOTAL_HEADCOUNT` − `TOTAL_TESTED` |
| `PARTICIPATION_PCT` | 82% | `TOTAL_TESTED / TOTAL_HEADCOUNT × 100` |

---

## Global: Per-Employee Health Score

Every page uses a composite health score per employee. Compute this once and reuse everywhere.

### Panel Status Assignment

For each employee, evaluate each of 8 diagnostic panels:

| Panel | At Risk | Monitor | Healthy |
|---|---|---|---|
| **Vitamin D** | `Vitamin D` < 20 ng/mL | 20–29 | ≥ 30 |
| **Lipid Profile** | ≥ 2 flags† | 1 flag | 0 flags |
| **CBC** | ≥ 2 flags‡ | 1 flag | 0 flags |
| **HbA1c / Diabetes** | `HbA1c` ≥ 6.5% OR `FBS` > 126 | `HbA1c` 5.7–6.4 OR `FBS` 110–126 | below thresholds |
| **Vitamin B12** | `VITAMIN B12` < 200 pg/mL | 200–299 | ≥ 300 |
| **Kidney** | ≥ 2 flags§ | 1 flag | 0 flags |
| **Thyroid** | `TSH` < 0.35 OR > 5.6 | `T3`/`T4` borderline | TSH 0.35–5.6 |
| **Liver** | `SGPT` > 40 AND `SGOT` > 50 | `SGPT` > 40 OR `SGOT` > 50 | both within range |

† Lipid flags: `Total Cholesterol` > 250, `Triglycerides` > 175, `Chol-LDL` > 100, `Chol-HDL` < 30
‡ CBC flags: `Hemoglobin` < 13.5 (Male) / < 12 (Female), `Platelet Count` < 150,000 or > 450,000, `WBC Morphology` abnormal — requires `Gender`
§ Kidney flags: `Serum Creatinine` > 1.4, `BUN` > 20, `Uric Acid` > 7.0 (Male) / > 6.0 (Female) — requires `Gender`

### Panel Score
```
At Risk  = 0 pts
Monitor  = 50 pts
Healthy  = 100 pts
```

### Weighted Composite Score per Employee
```
score = (Vitamin D × 0.15) + (Lipid × 0.20) + (CBC × 0.15) +
        (HbA1c × 0.15)    + (B12  × 0.10) + (Kidney × 0.10) +
        (Thyroid × 0.10)  + (Liver × 0.05)
```

### Risk Tier per Employee
```
abnormal_count = number of panels with status = At Risk
0 abnormal  → Low Risk
1–2         → Monitor
3+          → High Risk
```

### Organisation Averages (base = 449 tested)
```
Org health score  = AVG(score) across all tested employees = 78
Low Risk count    = COUNT(tier = Low)     = 278  (62% of 449)
Monitor count     = COUNT(tier = Monitor) = 117  (26% of 449)
High Risk count   = COUNT(tier = High)    =  54  (12% of 449)
```

---

## Page 1 — Dashboard (index.html)

All metrics recalculate on the active filter subset. Filters apply to `Dept`, `Gender`, `Age` (bucketed), `Location`.

---

### KPI 1 — Health Index

| Element | Value | Logic | Fields |
|---|---|---|---|
| Score | 78 | `AVG(employee_score)` across filtered employees | All 8 panel fields + `Gender` |
| Grade | A– | 75–100 = A, 60–74 = B, < 60 = C | derived from score |
| Gauge fill | 78% | `score / 100` | derived |

---

### KPI 2 — Evaluated

| Element | Value | Logic | Fields |
|---|---|---|---|
| Participation % | 82% | `TOTAL_TESTED / TOTAL_HEADCOUNT × 100` | `MRN` count, `TOTAL_HEADCOUNT` |
| Total Tested | 449 | `COUNT(rows in CSV)` | `MRN` |
| Not Tested | 99 | `TOTAL_HEADCOUNT − TOTAL_TESTED` | `TOTAL_HEADCOUNT` |
| Tested bar width | 82% | = Participation % | derived |
| Not Tested bar width | 18% | `TOTAL_NOT_TESTED / TOTAL_HEADCOUNT × 100` | derived |
| Gap note | "13% gap to participation target" | `100 − participation_pct` | derived |

---

### KPI 3 — Largest Risk

| Element | Value | Logic | Fields |
|---|---|---|---|
| % | 40% | `COUNT(metabolic_flag) / TOTAL_HEADCOUNT × 100` | see below |
| Label | "Metabolic Health" | Category with highest flagged % | derived |
| OPS sub-tile | 18% | `COUNT(metabolic_flag AND Dept=Operations) / COUNT(Dept=Operations) × 100` | `Dept` + metabolic fields |
| SALES sub-tile | 12% | same for Sales | |
| ENG sub-tile | 10% | same for Engineering | |

**Metabolic flag** (any one of):
```
Total Cholesterol > 250
Triglycerides     > 175
Chol-LDL          > 100
Fasting Blood Sugar (FBS) > 126
HbA1c             ≥ 6.5%
```
**Fields:** `Total Cholesterol`, `Triglycerides`, `Chol-LDL`, `Fasting Blood Sugar (FBS)`, `HbA1c`, `Dept`

---

### KPI 4 — Priority Action

| Element | Value | Logic | Fields |
|---|---|---|---|
| % | 28% | `COUNT(lipid_flag) / TOTAL_HEADCOUNT × 100` | lipid fields |
| Label | "Lipid Profile" | Panel with highest actionable flag % | derived |
| Gauge fill | 28% | = % | derived |

**Lipid flag** (any one of):
```
Total Cholesterol > 250
Triglycerides     > 175
Chol-LDL          > 100
Chol-HDL          < 30
```
**Fields:** `Total Cholesterol`, `Triglycerides`, `Chol-LDL`, `Chol-HDL`

---

### Department Health Ranking

| Dept | Score | Logic |
|---|---|---|
| Marketing | 84 | `AVG(employee_score WHERE Dept = Marketing)` |
| Engineering | 80 | `AVG(employee_score WHERE Dept = Engineering)` |
| Sales | 76 | `AVG(employee_score WHERE Dept = Sales)` |
| Operations | 68 | `AVG(employee_score WHERE Dept = Operations)` |

Bar width = score value. Benchmark line = 75 (org benchmark).

**Fields:** `Dept` + all 8 panel fields + `Gender`

---

### AI Workforce Summary

Narrative auto-generated from these computed facts:

| Insight | Logic | Fields |
|---|---|---|
| "Operations — 23% cardiovascular" | `COUNT(lipid_flag AND Dept=Ops) / COUNT(Dept=Ops)` | `Dept`, lipid fields |
| "Engineering — 65% Vitamin D" | `COUNT(VitD<20 AND Dept=Eng) / COUNT(Dept=Eng)` | `Dept`, `Vitamin D` |
| "Age >45 — 2× rate" | `abnormal_rate(Age>45) / abnormal_rate(Age≤45)` | `Age`, all panel fields |
| "Surat/Vadodara cluster" | Multi-param flag rate by `Location` | `Location`, `Vitamin D`, `Triglycerides`, `Fasting Blood Sugar` |

**Fields:** `Dept`, `Age`, `Location`, `Vitamin D`, lipid fields, `HbA1c`, `Fasting Blood Sugar`

---

### Aggregated Diagnostic Profiles (8 accordion rows)

`pct` formula for all rows: `COUNT(flag) / TOTAL_HEADCOUNT × 100`
Status: pct > 20% → At Risk · 10–20% → Monitor · < 10% → Healthy

| Row | pct | Flag definition | Fields |
|---|---|---|---|
| Vitamin D | 42% | `Vitamin D` < 20 ng/mL | `Vitamin D` |
| Lipid Profile | 28% | any lipid flag (see KPI 4) | `Total Cholesterol`, `Triglycerides`, `Chol-LDL`, `Chol-HDL` |
| Blood Counts (CBC) | 21% | any CBC flag (see panel table) | `Hemoglobin`, `Platelet Count (PLT)`, `WBC Morphology`, `Gender` |
| Diabetes (HbA1c) | 12% | `HbA1c` ≥ 5.7% OR `FBS` > 110 | `HbA1c`, `Fasting Blood Sugar (FBS)`, `Blood Sugar (Postprandial)` |
| Vitamin B12 | 15% | `VITAMIN B12` < 300 pg/mL | `VITAMIN B12` |
| Kidney Function | 3% | any kidney flag (see panel table) | `Serum Creatinine`, `BUN`, `Uric Acid`, `Gender` |
| Thyroid Profile | 4% | `TSH` < 0.35 OR > 5.6 | `TSH`, `T3 (Triiodothyronine)`, `T4 (Thyroxine)` |
| Liver Function | 5% | `SGPT` > 40 OR `SGOT` > 50 | `SGPT (ALT)`, `SGOT (AST)` |

Progress bar width = pct value.
AI Recommendation text in accordion = per-parameter metadata (not from CSV).

---

### Workforce Risk Profiling — Right Panel (3 tier cards)

**Tier counts (base = 449 tested):**

| Tier | % | Count | Logic |
|---|---|---|---|
| Low Clinical Risk | 62% | 278 | `COUNT(tier=Low)` |
| Monitoring Required | 26% | 117 | `COUNT(tier=Monitor)` |
| High Clinical Risk | 12% | 54 | `COUNT(tier=High)` |

**Breakdown rows within each card:**

```
Engineering Low %  = COUNT(Dept=Engineering AND tier=Low) / COUNT(Dept=Engineering) × 100  = 71%
Age <35 Low %      = COUNT(Age<35 AND tier=Low) / COUNT(Age<35) × 100                       = 68%
Female Low %       = COUNT(Gender=Female AND tier=Low) / COUNT(Gender=Female) × 100         = 65%

Sales Monitor %    = COUNT(Dept=Sales AND tier=Monitor) / COUNT(Dept=Sales) × 100           = 32%
Age 35-45 Mon %    = COUNT(Age 35-45 AND tier=Monitor) / COUNT(Age 35-45) × 100             = 29%
Hyderabad Mon %    = COUNT(Location=Hyderabad AND tier=Monitor) / COUNT(Location=Hyderabad) = 28%

Operations High %  = COUNT(Dept=Operations AND tier=High) / COUNT(Dept=Operations) × 100   = 23%
Age >45 High %     = COUNT(Age>45 AND tier=High) / COUNT(Age>45) × 100                      = 21%
Pune High %        = COUNT(Location=Pune AND tier=High) / COUNT(Location=Pune) × 100        = 18%
```

**Fields:** All 8 panel fields + `Gender`, `Dept`, `Age`, `Location`

---

## Page 2 — AI Insights (ai_insights.html)

---

### Hero Banner

| Element | Value | Logic | Fields |
|---|---|---|---|
| Overall Health | 78/100 | Same as Dashboard KPI 1 | All panel fields + `Gender` |
| High Risk % | 12% | `COUNT(tier=High) / TOTAL_TESTED × 100` | All panel fields + `Gender` |
| Priority Area | "Lipid Profile" | Panel with highest flagged % | All panel fields |

---

### Priority Risk Radar — 3 Gauge Cards

Uses **prevalence** (broader threshold than Clinical Profile) on base of 449 tested employees.

**Gauge 1 — Vitamin D**

| Element | Value | Logic | Fields |
|---|---|---|---|
| Gauge % | 68% | `COUNT(Vitamin D < 30) / 449 × 100` (includes borderline) | `Vitamin D` |
| Employee count | ~305 | `0.68 × 449` | derived |
| Sub-label dept | "Engineering" | `GROUP BY Dept → MAX(prevalence)` | `Dept`, `Vitamin D` |
| Badge | Critical | pct > 50% | derived |

**Gauge 2 — Lipid Profile**

| Element | Value | Logic | Fields |
|---|---|---|---|
| Gauge % | 34% | `COUNT(broader lipid flag†) / 449 × 100` | lipid fields |
| Sub-label | "Strongest cardiovascular risk driver" | static label | — |
| Badge | High Priority | derived from rank | — |

† Broader lipid flag: `Total Cholesterol` > 200 OR `Triglycerides` > 150 OR `Chol-LDL` > 100 OR `Chol-HDL` < 40

**Gauge 3 — Vitamin B12**

| Element | Value | Logic | Fields |
|---|---|---|---|
| Gauge % | 15% | `COUNT(VITAMIN B12 < 300) / 449 × 100` | `VITAMIN B12` |
| Sub-label age | "Aged 35–45" | `GROUP BY age_band → MAX(prevalence)` | `Age`, `VITAMIN B12` |
| Badge | Stable | pct < 20% | derived |

---

### Clinical Tier Profiles — 3 Cards

All averages computed on tested employees (449), grouped by tier.

**Card 1 — Low Risk (62%, 278 employees)**

| Marker | Avg Value | Logic | Fields |
|---|---|---|---|
| Vitamin D | 34 ng/mL | `AVG(Vitamin D WHERE tier=Low)` | `Vitamin D` |
| LDL Cholesterol | 88 mg/dL | `AVG(Chol-LDL WHERE tier=Low)` | `Chol-LDL` |
| HbA1c | 5.1% | `AVG(HbA1c WHERE tier=Low)` | `HbA1c` |
| Vitamin B12 | 382 pg/mL | `AVG(VITAMIN B12 WHERE tier=Low)` | `VITAMIN B12` |
| Hemoglobin | 14.2 g/dL | `AVG(Hemoglobin WHERE tier=Low)` | `Hemoglobin` |

Status dots: compare avg value to panel thresholds → all Normal for Low Risk tier.

**Card 2 — Monitor (26%, 117 employees)**

| Marker | Avg Value | Status |
|---|---|---|
| Vitamin D | 17 ng/mL | Borderline (< 20) |
| LDL Cholesterol | 114 mg/dL | Borderline (> 100) |
| HbA1c | 5.9% | Borderline (5.7–6.4) |
| Vitamin B12 | 241 pg/mL | Borderline (200–299) |
| Hemoglobin | 13.0 g/dL | Borderline (< 13.5) |

**Card 3 — High Risk (12%, 54 employees)**

| Marker | Avg Value | Status |
|---|---|---|
| Vitamin D | 8 ng/mL | Abnormal (< 20) |
| LDL Cholesterol | 152 mg/dL | Abnormal (> 100) |
| HbA1c | 6.8% | Abnormal (≥ 6.5) |
| Vitamin B12 | 164 pg/mL | Abnormal (< 200) |
| Hemoglobin | 11.4 g/dL | Abnormal (< 12) |

**Fields for all 3 cards:** `Vitamin D`, `Chol-LDL`, `HbA1c`, `VITAMIN B12`, `Hemoglobin` + all tier computation fields

---

### Deep Dive Health Intelligence — 3 Tabs

All pcts use base = 449 tested employees.

**Tab: Vitamin D**

| Element | Value | Logic | Fields |
|---|---|---|---|
| critical_pct | 5% | `COUNT(Vitamin D < 10) / 449 × 100` | `Vitamin D` |
| at_risk_pct (donut) | 42% | `COUNT(Vitamin D < 20) / 449 × 100` | `Vitamin D` |
| AI insight ratio | "2.4×" | `(COUNT(VitD<20 AND Dept=Eng) / COUNT(Dept=Eng)) / (COUNT(VitD<20) / 449)` | `Vitamin D`, `Dept` |
| Diet Tips (3 cards) | text | per-parameter metadata — not from CSV | — |
| Complications (3 pills) | text | per-parameter metadata — not from CSV | — |
| HR Actions (2 cards) | text + impact | per-parameter metadata — not from CSV | — |

**Tab: Lipid Profile**

| Element | Value | Logic | Fields |
|---|---|---|---|
| critical_pct | 12% | `COUNT(Chol-LDL > 160 OR Total Cholesterol > 300) / 449 × 100` | `Chol-LDL`, `Total Cholesterol` |
| at_risk_pct (donut) | 31% | `COUNT(any lipid flag) / 449 × 100` | all lipid fields |
| AI insight dept | "Sales" | `GROUP BY Dept → highest lipid flag %` | `Dept`, lipid fields |

**Tab: Vitamin B12**

| Element | Value | Logic | Fields |
|---|---|---|---|
| critical_pct | 8% | `COUNT(VITAMIN B12 < 150) / 449 × 100` | `VITAMIN B12` |
| at_risk_pct (donut) | 15% | `COUNT(VITAMIN B12 < 300) / 449 × 100` | `VITAMIN B12` |
| AI insight cohort | "Gen-Z" | `GROUP BY Age < 27 → B12 flag rate` | `Age`, `VITAMIN B12` |

---

### Intervention Simulator

| Element | Value | Logic | Fields |
|---|---|---|---|
| Base score | 78 | Org health index (KPI 1) | All panel fields |
| Toggle 1 — Vitamin D | +3 pts | Editorial estimate | — |
| Toggle 2 — Lipid Profile | +4 pts | Editorial estimate | — |
| Toggle 3 — HbA1c | +3 pts | Editorial estimate | — |
| Projected score | 78 + selected | Sum of active toggle values | derived |
| Insurance saving | $12,400 | Editorial actuarial estimate | — |
| Model base | 548 employees | `TOTAL_HEADCOUNT` | `TOTAL_HEADCOUNT` |

---

### AI-Discovered Risk Clusters

| Cluster | Definition | Fields |
|---|---|---|
| "The Red-Eye Hub" | `Dept = Operations AND Vitamin D < 20 AND FBS > 110` | `Dept`, `Vitamin D`, `Fasting Blood Sugar` |
| "Sedentary Spikes" | `Dept = Legal AND Triglycerides > 150 AND BMI < 25` | `Dept`, `Triglycerides`, `BMI` |
| "Nutrient Gaps" | `Age < 27 AND VITAMIN B12 < 200` | `Age`, `VITAMIN B12` |

Status badges (Rising / Watch / Stable) = derived from direction of cluster's primary metric vs org average.

---

## Page 3 — Workforce (workforce.html)

---

### Page Hero

| Element | Value | Logic | Fields |
|---|---|---|---|
| Total employees | 548 | `TOTAL_HEADCOUNT` | constant |
| Dept filter options | Operations, Engineering, Sales, Marketing, HR & Legal | `DISTINCT(Dept)` | `Dept` |

---

### KPI Overview — 4 Cards

**Low Risk (62%, 278 employees)**

| Element | Value | Logic |
|---|---|---|
| % | 62% | `COUNT(tier=Low) / TOTAL_TESTED × 100` |
| Employee count | 278 | `COUNT(tier=Low)` |
| Bar width | 62% | = % |

**Monitoring Required (26%, 117 employees)**

| Element | Value | Logic |
|---|---|---|
| % | 26% | `COUNT(tier=Monitor) / TOTAL_TESTED × 100` |
| Employee count | 117 | `COUNT(tier=Monitor)` |

**High Risk (12%, 54 employees)**

| Element | Value | Logic |
|---|---|---|
| % | 12% | `COUNT(tier=High) / TOTAL_TESTED × 100` |
| Employee count | 54 | `COUNT(tier=High)` |

**Avg Wellness Score (78/100)**

| Element | Value | Logic |
|---|---|---|
| Score | 78 | `AVG(employee_score)` — same as Dashboard KPI 1 |
| Grade | A– | 75–100 = A, 60–74 = B, < 60 = C |

**Fields for all 4 cards:** All 8 panel fields + `Gender`

---

### Risk Distribution Chart

| Element | Logic | Fields |
|---|---|---|
| X-axis | Health score 0–100 | derived |
| Y-axis | Count of employees at each score band | derived |
| Curve | Plot `employee_score` distribution as smooth curve | all panel fields + `Gender` |
| Peak annotation | Score range with highest employee density | derived |
| "Left-skewed" note | 38% of employees score < 75 = `COUNT(score<75) / TOTAL_TESTED` | derived |
| Zone fills | Low (0–59), Mid (60–74), High (75–100) — count employees per zone | derived |

**Fields:** All 8 panel fields + `Gender`

---

### Priority Interventions — 2 Alert Cards

**Alert 1 — Operations Risk Spike**

| Element | Value | Logic | Fields |
|---|---|---|---|
| Dept name | Operations | `GROUP BY Dept → highest lipid spike` | `Dept`, lipid fields |
| "40% surge" | 40% | `COUNT(lipid_flag AND Dept=Operations) / COUNT(Dept=Operations) × 100` | `Dept`, lipid fields |
| Affected employees | ~54 | `COUNT(tier=High AND Dept=Operations)` | `Dept` + tier |

**Alert 2 — Engineering Deficiencies**

| Element | Value | Logic | Fields |
|---|---|---|---|
| Dept name | Engineering | `GROUP BY Dept → highest VitD deficiency` | `Dept`, `Vitamin D` |
| "65%" | 65% | `COUNT(VitD<20 AND Dept=Engineering) / COUNT(Dept=Engineering) × 100` | `Dept`, `Vitamin D` |
| Affected employees | ~98 | `COUNT(Dept=Engineering) × deficiency_rate ≈ 150 × 0.65 ≈ 98` | `Dept`, `Vitamin D` |

**Workforce stats footer:**

| Element | Value | Logic |
|---|---|---|
| Employees Assessed | 548 | `TOTAL_HEADCOUNT` |
| Depts Profiled | 5 | `COUNT(DISTINCT Dept)` |

---

### Productivity Risk Map (Bubble Chart)

| Element | Logic | Fields |
|---|---|---|
| Y-axis (headcount) | `COUNT(Dept=X)` per department | `Dept` |
| Bubble size | `AVG(employee_score WHERE Dept=X)` — larger = higher avg risk score | all panel fields + `Gender`, `Dept` |
| X-axis position | Editorial (difficulty of intervention) | — |
| Quadrant labels | Static (High Concern / Scale Risk / Low Impact / Watch) | — |

---

### AI-Generated Intervention Personas — 3 Cards

**Persona 1 — "The Sedentary Operator" (14%, ~63 employees)**

| Element | Value | Logic | Fields |
|---|---|---|---|
| 14% of workforce | 63 employees | `COUNT(persona_1_flag) / TOTAL_TESTED × 100` | see definition |
| Predominantly in | Operations | `GROUP BY Dept → highest persona concentration` | `Dept` |
| Avg risk score | 52 | `AVG(employee_score WHERE persona_1)` | all panel fields |
| Elevated LDL bar | 68% | `COUNT(Chol-LDL>100 AND persona_1) / COUNT(persona_1) × 100` | `Chol-LDL` |
| High Uric Acid bar | 42% | `COUNT(Uric_Acid>7 AND persona_1) / COUNT(persona_1) × 100` | `Uric Acid` |
| Elevated HbA1c bar | 31% | `COUNT(HbA1c>5.7 AND persona_1) / COUNT(persona_1) × 100` | `HbA1c` |

Persona 1 definition: `Dept=Operations AND Chol-LDL > 100 AND HbA1c > 5.7`

**Persona 2 — "The Depleted Developer" (22%, ~99 employees)**

| Element | Value | Logic | Fields |
|---|---|---|---|
| 22% of workforce | 99 employees | `COUNT(persona_2_flag) / TOTAL_TESTED × 100` | see definition |
| Predominantly in | Engineering | `GROUP BY Dept → highest persona concentration` | `Dept` |
| Avg risk score | 68 | `AVG(employee_score WHERE persona_2)` | all panel fields |
| Vitamin D Deficit bar | 81% | `COUNT(VitD<20 AND persona_2) / COUNT(persona_2) × 100` | `Vitamin D` |
| B12 Low bar | 55% | `COUNT(B12<300 AND persona_2) / COUNT(persona_2) × 100` | `VITAMIN B12` |

Persona 2 definition: `Dept=Engineering AND Vitamin D < 20 AND VITAMIN B12 < 300`

**Persona 3 — "The Stressed Achiever" (implied ~64% remaining in Sales/Marketing)**

Persona 3 definition: `Dept=Sales AND Triglycerides > 150 AND HbA1c > 5.7`
Fields: `Dept`, `Triglycerides`, `HbA1c`

---

### Parameter Risk Intelligence — Tab 1: By Department

For each parameter, for each department:
```
dept_pct(param, dept) = COUNT(param_flag AND Dept=dept) / COUNT(Dept=dept) × 100
```

| Parameter | Dept | % | Primary field |
|---|---|---|---|
| Vitamin D | ENG | 65% | `Vitamin D` < 20 |
| | OPS | 58% | |
| | SALES | 31% | |
| | HR/LGL | 22% | |
| | MKT | 18% | |
| Lipid Profile | OPS | 40% | any lipid flag |
| | SALES | 35% | |
| | ENG | 22% | |
| | MKT | 14% | |
| | HR/LGL | 8% | |
| CBC / Anaemia | OPS | 24% | any CBC flag |
| | SALES | 21% | |
| | ENG | 19% | |
| | MKT | 15% | |
| | HR/LGL | 11% | |
| Vitamin B12 | ENG | 24% | `VITAMIN B12` < 300 |
| | MKT | 18% | |
| | SALES | 16% | |
| | OPS | 12% | |
| | HR/LGL | 10% | |
| HbA1c | OPS | 18% | `HbA1c` ≥ 5.7 |
| | SALES | 15% | |
| | ENG | 8% | |
| | MKT | 6% | |
| | HR/LGL | 4% | |

Bars sorted highest → lowest per parameter. Org avg label = `COUNT(param_flag) / TOTAL_HEADCOUNT × 100`.

**Fields:** `Dept` + each parameter's flag field(s) + `Gender` (for CBC/Kidney thresholds)

---

### Parameter Risk Intelligence — Tab 2: By Gender

```
female_pct(param) = COUNT(param_flag AND Gender=Female) / COUNT(Gender=Female) × 100
male_pct(param)   = COUNT(param_flag AND Gender=Male)   / COUNT(Gender=Male)   × 100
gap_pp            = female_pct − male_pct  (positive = female-skewed, negative = male-skewed)
```

| Parameter | Female | Male | Gap | Note |
|---|---|---|---|---|
| Vitamin D | 48% | 38% | F+10pp | — |
| Lipid Profile | 22% | 33% | M+11pp | male-skewed |
| CBC / Anaemia | 34% | 11% | F+23pp | **Biggest gap** — anaemia is female-skewed |
| Vitamin B12 | 18% | 13% | F+5pp | — |
| HbA1c | 9% | 15% | M+6pp | male-skewed |

"Biggest gap" badge = parameter with largest absolute `|gap_pp|`.

**Fields:** `Gender` + each parameter's flag field(s)

---

### Parameter Risk Intelligence — Tab 3: Co-occurrence Matrix

```
cell(A, B) = COUNT(flagA AND flagB) / COUNT(flagA) × 100
```

i.e. "of employees flagged for A, what % are also flagged for B?"

| | VitD | Lipid | CBC | B12 | HbA1c |
|---|---|---|---|---|---|
| **VitD** | — | 41% | 19% | 33% | 29% |
| **Lipid** | 41% | — | 16% | 12% | 38% |
| **CBC** | 19% | 16% | — | 24% | 14% |
| **B12** | 33% | 12% | 24% | — | 11% |
| **HbA1c** | 29% | 38% | 14% | 11% | — |

Cell colour intensity = `rgba(113,42,226, value/150)` — purple tint proportional to co-occurrence %.

Key insight: Vit D ↔ Lipid (41%) and Lipid ↔ HbA1c (38%) are the strongest co-occurrences.

**Fields:** `Vitamin D`, lipid fields, CBC fields, `VITAMIN B12`, `HbA1c`, `Gender`

---

### Intervention Prioritization Matrix

Editorial 2×2 grid (Impact vs Effort). Point estimates are not computed from CSV.

| Quadrant | Items | Point estimate |
|---|---|---|
| Quick Wins (High Impact, Low Effort) | Vitamin D Screening | +3.2 pts |
| | B12 Awareness Drive | +1.8 pts |
| Strategic Bets (High Impact, High Effort) | Lipid Management Program | +4.1 pts |
| | Stress Reduction Program | +2.4 pts |
| Minor Tasks (Low Impact, Low Effort) | Hydration Campaign | +0.8 pts |
| De-prioritize (Low Impact, High Effort) | Annual Health Drive | — |

No CSV fields required — purely editorial.

---

### High-Risk Cohort Explorer

**Summary cards:**

| Element | Value | Logic | Fields |
|---|---|---|---|
| Total High-Risk | 54 | `COUNT(tier=High)` | all panel fields + `Gender` |
| Highest Dept % | 23% | `MAX(COUNT(tier=High AND Dept=X) / COUNT(Dept=X))` | `Dept` + tier |
| Lowest Dept % | 5% | `MIN(same formula)` | `Dept` + tier |
| Active Alerts | 3 | Editorial count of flagged cohorts | — |

**Table rows:**

| Cohort | Headcount | % At Risk | Primary Risk Driver | Logic |
|---|---|---|---|---|
| Operations Team | 100 | 23% | Lipid Profile | `COUNT(Dept=Ops)` = 100 · `COUNT(tier=High AND Dept=Ops)/100` = 23% · most prevalent flag in Ops |
| Employees > 45 | 87 | 21% | HbA1c | `COUNT(Age>45)` = 87 · `COUNT(tier=High AND Age>45)/87` = 21% |
| Engineering | 150 | 18% | Vitamin D | `COUNT(Dept=Eng)` = 150 · `COUNT(tier=High AND Dept=Eng)/150` = 18% |
| Pune Office | 72 | 18% | Multi-factor | `COUNT(Location=Pune)` = 72 · `COUNT(tier=High AND Location=Pune)/72` = 18% |
| Sales Force | 80 | 10% | HbA1c mild | `COUNT(Dept=Sales)` = 80 · `COUNT(tier=High AND Dept=Sales)/80` = 10% |
| HR & Legal | 38 | 5% | Vitamin D | `COUNT(Dept=HR)` = 38 · `COUNT(tier=High AND Dept=HR)/38` = 5% |

Primary Risk Driver per cohort = `MODE(most_flagged_panel WHERE cohort AND tier=High)`

Priority badge logic:
- Urgent: dept risk % > 20%
- Monitor: dept risk % 10–20%
- Plan / Watch: dept risk % < 10%

**Fields:** `Dept`, `Age`, `Location` + all panel fields + `Gender`

---

## Complete Field Index

### Employee CSV Fields (68 total)

**Identity & Demographics (6)**
| # | Field | Used in |
|---|---|---|
| 1 | `MRN` | KPI 2 (tested count) |
| 2 | `Patient Name` | — (display only) |
| 3 | `Age` | Age filters, AI Insights B12 insight, Cohort Explorer |
| 4 | `Gender` | CBC thresholds, Kidney thresholds, Gender tab |
| 5 | `Dept` | All dept breakdowns, rankings, personas, clusters |
| 6 | `Location` | Risk Profiling location breakdowns, AI Summary geo-insight |

**New fields to add (2)**
| # | Field | Used in |
|---|---|---|
| 7 | `HbA1c` | Diabetes panel, KPI 3 metabolic flag, Clinical Tier Profiles, Personas |
| 8 | `BMI` | AI Clusters "Sedentary Spikes" definition |

**CBC / Blood Counts (17)**
`Hemoglobin`, `Total RBC Count`, `Packed Cell Volume (PCV)`, `MCV`, `MCH`, `MCHC`, `RDW`, `RBC Count`, `WBC Morphology`, `Neutrophils`, `Lymphocytes`, `Monocytes`, `Eosinophils`, `Basophils`, `Platelet Count (PLT)`, `MPV`, `ESR`

Of these, **flag-driving fields for panels:** `Hemoglobin`, `Platelet Count (PLT)`, `WBC Morphology`
Other CBC fields used for detailed clinical reporting only.

**Blood Sugar (2)**
`Fasting Blood Sugar (FBS)`, `Blood Sugar (Postprandial)`

**Lipid Profile (8)**
`Total Cholesterol`, `Triglycerides`, `Chol-VLDL`, `Chol-LDL`, `Chol-HDL`, `Total Cholesterol : HDL ratio`, `LDL : HDL Ratio`, `Lipid Profile-TOTAL LIPID`

Of these, **flag-driving:** `Total Cholesterol`, `Triglycerides`, `Chol-LDL`, `Chol-HDL`

**Kidney Function (3)**
`Serum Creatinine`, `BUN`, `Uric Acid`

**Liver Function (2)**
`SGPT (ALT)`, `SGOT (AST)`

**Thyroid (3)**
`T3 (Triiodothyronine)`, `T4 (Thyroxine)`, `TSH`

**Vitamins (2)**
`Vitamin D`, `VITAMIN B12`

**Urine Analysis (17)**
`Colour`, `Odour`, `Appearence`, `PH`, `Specific Gravity`, `URINE SPOT PROTEIN`, `Glucose in Urine`, `Ketone`, `Bilirubin`, `Nitrite`, `Leukocytes`, `Blood in Urine`, `Pus Cells`, `RBC`, `Casts`, `Crystals`, `Epithelial Cells`

Note: Urine fields are not currently used in any dashboard computation. They are available for future clinical expansion (e.g. Kidney Function deeper analysis).

**Other (1)**
`PSA` — Prostate Specific Antigen. Male-only field, not currently used in any dashboard panel.

---

### External / Non-CSV Data

| Constant / Metadata | Value | Where used |
|---|---|---|
| `TOTAL_HEADCOUNT` | 548 | KPI 2, Workforce hero, Intervention Simulator |
| Per-parameter tips (3 per param) | Editorial | Deep Dive Diet & Lifestyle Tips |
| Per-parameter complications (3 per param) | Editorial | Deep Dive Clinical Complications |
| Per-parameter HR actions (2 per param) | Editorial | Deep Dive Strategic HR Interventions |
| Intervention point estimates (+3/+4/+3) | Editorial | Intervention Simulator toggles |
| Insurance saving ($12,400) | Editorial actuarial | Intervention Simulator |
| Intervention Prioritization Matrix positions | Editorial | Workforce Intervention Matrix |
| AI Cluster definitions | Editorial | AI Clusters, AI Personas |
