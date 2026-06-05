# NirogGyan — Data Fields, Sample Employees & Full Computation Walkthrough

This document covers:
1. Every field required to power the dashboard, with source and reference range
2. Five sample employee records with all 68 fields populated
3. Full step-by-step computation of every dashboard number using those 5 employees

---

## Part 1 — Complete Field Catalogue (68 Employee Fields)

### Legend
- **CSV** = already present in gujrat-excel.csv
- **ADD** = new field to be added to the CSV
- **Flag field** = directly used in panel status logic
- **Support field** = present in CSV, available for clinical detail but not a primary flag driver

---

### Group A — Identity & Demographics

| # | Field | Type | Source | Reference / Values | Used In |
|---|---|---|---|---|---|
| 1 | `MRN` | String | CSV | Unique patient/employee ID | KPI 2 tested count |
| 2 | `Patient Name` | String | CSV | Full name | Display only |
| 3 | `Age` | String | CSV | e.g. "49 Yrs" | Age-group filter, Cohort Explorer, AI Insight |
| 4 | `Gender` | String | CSV | Male / Female | CBC & Kidney flag thresholds, Gender tab |
| 5 | `Dept` | String | CSV *(fix values)* | Engineering / Sales / Operations / Marketing / HR & Legal | All dept breakdowns |
| 6 | `Location` | String | **ADD** | Bengaluru / Surat / Hyderabad / Pune / Mumbai | Location filter, AI Summary geo-insight |
| 7 | `BMI` | Numeric | **ADD** | kg/m² · Normal 18.5–24.9 | AI Cluster "Sedentary Spikes" |
| 8 | `HbA1c` | Numeric | **ADD** | % · Normal < 5.7 · Pre-diabetic 5.7–6.4 · Diabetic ≥ 6.5 | Diabetes panel, KPI 3 metabolic flag, Clinical Tier Profiles |

---

### Group B — CBC / Blood Counts (17 fields)

| # | Field | Type | Source | Reference Range | Flag? |
|---|---|---|---|---|---|
| 9 | `Hemoglobin (HB%)` | Numeric | CSV | Male ≥ 13.5 g/dL · Female ≥ 12.0 g/dL | **Flag** (needs `Gender`) |
| 10 | `Total RBC Count` | Numeric | CSV | 3.5–5.5 × 10⁶/µL | Support |
| 11 | `Packed Cell Volume (PCV)` | Numeric | CSV | 35–55 % | Support |
| 12 | `MCV` | Numeric | CSV | 78–98 fL | Support |
| 13 | `MCH` | Numeric | CSV | 28–35 pg | Support |
| 14 | `MCHC` | Numeric | CSV | 31–36 g/dL | Support |
| 15 | `RDW` | Numeric | CSV | 11.5–14.5 % | Support |
| 16 | `RBC Count` | Numeric | CSV | Reported value | Support |
| 17 | `WBC Morphology` | Numeric | CSV | 4,000–11,000 /µL | **Flag** (outside range) |
| 18 | `Neutrophils` | Numeric | CSV | 45–75 % | Support |
| 19 | `Lymphocytes` | Numeric | CSV | 25–45 % | Support |
| 20 | `Monocytes` | Numeric | CSV | 3–8 % | Support |
| 21 | `Eosinophils` | Numeric | CSV | 1–6 % | Support |
| 22 | `Basophils` | Numeric | CSV | 0–1 % | Support |
| 23 | `Abs. Neutrophil Count` | Numeric | CSV | 1,800–7,500 /µL | Support |
| 24 | `Abs. Lymphocyte Count` | Numeric | CSV | 1,000–4,800 /µL | Support |
| 25 | `Abs. Monocyte Count` | Numeric | CSV | 200–1,000 /µL | Support |
| 26 | `Abs. Eosinophil Count` | Numeric | CSV | 100–500 /µL | Support |
| 27 | `Abs. Basophil Count` | Numeric | CSV | 0–100 /µL | Support |
| 28 | `Platelet Count (PLT)` | Numeric | CSV | 150,000–450,000 /µL | **Flag** |
| 29 | `MPV` | Numeric | CSV | 7.5–12.5 fL | Support |
| 30 | `ESR` | Numeric | CSV | 1–10 mm/hr | Support |

**CBC Panel flag rule:** flag if `Hemoglobin` below threshold (gender-specific) OR `Platelet Count` outside 150k–450k OR `WBC Morphology` outside 4,000–11,000.
Panel status: ≥ 2 flags → At Risk · 1 flag → Monitor · 0 flags → Healthy

---

### Group C — Blood Sugar (2 fields)

| # | Field | Type | Source | Reference Range | Flag? |
|---|---|---|---|---|---|
| 31 | `Fasting Blood Sugar (FBS)` | Numeric | CSV | 70–110 mg/dL | **Flag** (> 110 Monitor, > 126 At Risk) |
| 32 | `Blood Sugar (Postprandial)` | Numeric | CSV | < 140 mg/dL | Support |

---

### Group D — Lipid Profile (8 fields)

| # | Field | Type | Source | Reference Range | Flag? |
|---|---|---|---|---|---|
| 33 | `Total Cholesterol` | Numeric | CSV | 150–250 mg/dL | **Flag** (> 250) |
| 34 | `Triglycerides (TGL)` | Numeric | CSV | 30–175 mg/dL | **Flag** (> 175) |
| 35 | `Chol-VLDL` | Numeric | CSV | 10–30 mg/dL | Support |
| 36 | `Chol-LDL` | Numeric | CSV | 0–100 mg/dL | **Flag** (> 100) |
| 37 | `Chol-HDL` | Numeric | CSV | 30–50 mg/dL | **Flag** (< 30) |
| 38 | `Total Cholesterol : HDL ratio` | Numeric | CSV | < 5.0 | Support |
| 39 | `LDL : HDL Ratio` | Numeric | CSV | < 3.5 | Support |
| 40 | `Lipid Profile-TOTAL LIPID` | Numeric | CSV | Reported value | Support |

**Lipid Panel flag rule:** count flags from `Total Cholesterol` > 250, `Triglycerides` > 175, `Chol-LDL` > 100, `Chol-HDL` < 30.
Panel status: ≥ 2 flags → At Risk · 1 flag → Monitor · 0 flags → Healthy

---

### Group E — Kidney Function (3 fields)

| # | Field | Type | Source | Reference Range | Flag? |
|---|---|---|---|---|---|
| 41 | `Serum Creatinine` | Numeric | CSV | 0.4–1.4 mg/dL | **Flag** (> 1.4) |
| 42 | `BUN` | Numeric | CSV | 7–20 mg/dL | **Flag** (> 20) |
| 43 | `Uric Acid` | Numeric | CSV | Male < 7.0 · Female < 6.0 mg/dL | **Flag** (needs `Gender`) |

**Kidney Panel flag rule:** count flags from the 3 fields above (thresholds gender-adjusted for Uric Acid).
Panel status: ≥ 2 flags → At Risk · 1 flag → Monitor · 0 flags → Healthy

---

### Group F — Liver Function (2 fields)

| # | Field | Type | Source | Reference Range | Flag? |
|---|---|---|---|---|---|
| 44 | `SGPT (ALT)` | Numeric | CSV | 5–40 IU/L | **Flag** (> 40) |
| 45 | `SGOT (AST)` | Numeric | CSV | 5–50 IU/L | **Flag** (> 50) |

**Liver Panel flag rule:** both `SGPT` > 40 AND `SGOT` > 50 → At Risk · either one elevated → Monitor · both normal → Healthy

---

### Group G — Thyroid Profile (3 fields)

| # | Field | Type | Source | Reference Range | Flag? |
|---|---|---|---|---|---|
| 46 | `T3 (Triiodothyronine)` | Numeric | CSV | 0.8–2.0 ng/mL | Support |
| 47 | `T4 (Thyroxine)` | Numeric | CSV | 5.1–14.1 µg/dL | Support |
| 48 | `TSH` | Numeric | CSV | 0.35–5.6 µIU/mL | **Flag** (outside range) |

**Thyroid Panel flag rule:** `TSH` < 0.35 OR > 5.6 → At Risk · `T3` or `T4` borderline while TSH marginal → Monitor · all normal → Healthy

---

### Group H — Vitamins (2 fields)

| # | Field | Type | Source | Reference Range | Flag? |
|---|---|---|---|---|---|
| 49 | `Vitamin D` | Numeric | CSV | ≥ 30 ng/mL sufficient · 20–29 insufficient · < 20 deficient | **Flag** (< 20 = At Risk · 20–29 = Monitor) |
| 50 | `VITAMIN B12` | Numeric | CSV | ≥ 300 pg/mL normal · 200–299 borderline · < 200 deficient | **Flag** (< 200 = At Risk · 200–299 = Monitor) |

---

### Group I — Urine Analysis (17 fields)

All fields from CSV. Currently not used in any panel computation — available for future Kidney Function expansion.

| # | Field | Values |
|---|---|---|
| 51 | `Colour` | Pale Yellow / Yellow / Amber |
| 52 | `Odour` | Non Specific |
| 53 | `Appearence` | Clear / Turbid |
| 54 | `PH` | 4.5–8.0 |
| 55 | `Specific Gravity` | 1.001–1.030 |
| 56 | `URINE SPOT PROTEIN` | Nil / Trace / Present |
| 57 | `Glucose in Urine` | Nil / Present |
| 58 | `Ketone` | Nil / Present |
| 59 | `Bilirubin` | Nil / Present |
| 60 | `Nitrite` | Nil / Present |
| 61 | `Leukocytes` | Nil / Present |
| 62 | `Blood in Urine` | Nil / Present |
| 63 | `Pus Cells` | Nil / 0–5 |
| 64 | `RBC` | Nil / 0–2 |
| 65 | `Casts` | Nil |
| 66 | `Crystals` | Nil |
| 67 | `Epithelial Cells` | Nil / Few |

---

### Group J — Other (1 field)

| # | Field | Type | Source | Reference | Notes |
|---|---|---|---|---|---|
| 68 | `PSA` | Numeric | CSV | < 4.0 ng/mL | Male only. Not used in current panel logic. |

---

## Part 2 — Five Sample Employees (All 68 Fields)

Five employees chosen to represent the full risk spectrum. Two are High Risk, two are Monitor, one is Low Risk.

---

### Employee 1 — PRIYA SHARMA · Low Risk

| Field | Value | Status |
|---|---|---|
| **MRN** | EMP001 | — |
| **Patient Name** | Ms. PRIYA SHARMA | — |
| **Age** | 28 Yrs | — |
| **Gender** | Female | — |
| **Dept** | Engineering | — |
| **Location** | Bengaluru | — |
| **BMI** | 21.5 | Normal |
| **HbA1c** | 5.0% | Normal |
| Hemoglobin | 13.5 g/dL | Normal (F ≥ 12) |
| Total RBC Count | 4.5 | Normal |
| PCV | 41.0 | Normal |
| MCV | 87.0 | Normal |
| MCH | 30.2 | Normal |
| MCHC | 33.8 | Normal |
| RDW | 13.0 | Normal |
| RBC Count | 0 | — |
| WBC Morphology | 6200 | Normal |
| Neutrophils | 58.0 | Normal |
| Lymphocytes | 34.0 | Normal |
| Monocytes | 5.0 | Normal |
| Eosinophils | 2.5 | Normal |
| Basophils | 0.5 | Normal |
| Abs. Neutrophil Count | 3596 | Normal |
| Abs. Lymphocyte Count | 2108 | Normal |
| Abs. Monocyte Count | 310 | Normal |
| Abs. Eosinophil Count | 155 | Normal |
| Abs. Basophil Count | 31 | Normal |
| Platelet Count (PLT) | 275000 | Normal |
| MPV | 10.2 | Normal |
| ESR | 8 | Normal |
| Fasting Blood Sugar (FBS) | 82 | Normal |
| Blood Sugar (Postprandial) | 108 | Normal |
| Total Cholesterol | 175 | Normal |
| Triglycerides | 95 | Normal |
| Chol-VLDL | 19 | Normal |
| Chol-LDL | 85 | Normal ✓ |
| Chol-HDL | 52 | Normal |
| TC : HDL ratio | 3.37 | Normal |
| LDL : HDL Ratio | 1.63 | Normal |
| Lipid Profile-TOTAL LIPID | 438 | Normal |
| Serum Creatinine | 0.72 | Normal |
| BUN | 11.2 | Normal |
| Uric Acid | 4.1 | Normal (F < 6.0) |
| SGPT (ALT) | 18 | Normal |
| SGOT (AST) | 20 | Normal |
| T3 | 1.21 | Normal |
| T4 | 8.2 | Normal |
| TSH | 2.34 | Normal |
| Colour | Pale Yellow | — |
| Odour | Non Specific | — |
| Appearence | Clear | — |
| PH | 6.0 | Normal |
| Specific Gravity | 1.012 | Normal |
| URINE SPOT PROTEIN | Nil | Normal |
| Glucose in Urine | Nil | Normal |
| Ketone | Nil | Normal |
| Bilirubin | Nil | Normal |
| Nitrite | Nil | Normal |
| Leukocytes | Nil | Normal |
| Blood in Urine | Nil | Normal |
| Pus Cells | Nil | Normal |
| RBC | Nil | Normal |
| Casts | Nil | Normal |
| Crystals | Nil | Normal |
| Epithelial Cells | Nil | Normal |
| **Vitamin D** | **34.0** | **Healthy (≥ 30)** |
| **VITAMIN B12** | **412** | **Healthy (≥ 300)** |
| PSA | N/A | Female |

---

### Employee 2 — JOTHISHKUMAR M.S. · Monitor

*(Source: MRN 2045200076 from CSV, Dept + Location + HbA1c + BMI added)*

| Field | Value | Status |
|---|---|---|
| **MRN** | 2045200076 | — |
| **Patient Name** | Mr. JOTHISHKUMAR M.S. | — |
| **Age** | 49 Yrs | — |
| **Gender** | Male | — |
| **Dept** | Operations | — |
| **Location** | Surat | — |
| **BMI** | 26.4 | Overweight |
| **HbA1c** | 5.4% | Normal |
| Hemoglobin | 14.9 | Normal (M ≥ 13.5) |
| Total RBC Count | 5.03 | Normal |
| PCV | 44.6 | Normal |
| MCV | 88.7 | Normal |
| MCH | 29.7 | Normal |
| MCHC | 33.5 | Normal |
| RDW | 13.2 | Normal |
| RBC Count | 0 | — |
| WBC Morphology | 5590 | Normal |
| Neutrophils | 59.1 | Normal |
| Lymphocytes | 31.1 | Normal |
| Monocytes | 6.7 | Normal |
| Eosinophils | 2.8 | Normal |
| Basophils | 0.3 | Normal |
| Abs. Neutrophil Count | 3300 | Normal |
| Abs. Lymphocyte Count | 1740 | Normal |
| Abs. Monocyte Count | 370 | Normal |
| Abs. Eosinophil Count | 160 | Normal |
| Abs. Basophil Count | 20 | Normal |
| Platelet Count (PLT) | 289000 | Normal |
| MPV | 10.0 | Normal |
| ESR | 21 | Slightly elevated |
| Fasting Blood Sugar (FBS) | 89.88 | Normal |
| Blood Sugar (Postprandial) | 104.64 | Normal |
| Total Cholesterol | 212.6 | Normal |
| Triglycerides | 88.2 | Normal |
| Chol-VLDL | 18.0 | Normal |
| Chol-LDL | 150.6 | **FLAGGED** (> 100) |
| Chol-HDL | 44.0 | Normal |
| TC : HDL ratio | 4.83 | Normal |
| LDL : HDL Ratio | 3.42 | Normal |
| Lipid Profile-TOTAL LIPID | 561.6 | — |
| Serum Creatinine | 0.9 | Normal |
| BUN | 10.3 | Normal |
| Uric Acid | 5.42 | Normal (M < 7.0) |
| SGPT (ALT) | 39 | Normal |
| SGOT (AST) | 26.37 | Normal |
| T3 | 0.99 | Normal |
| T4 | 6.7 | Normal |
| TSH | 2.02 | Normal |
| Colour | Pale Yellow | — |
| Odour | Non Specific | — |
| Appearence | Clear | — |
| PH | 5.0 | Normal |
| Specific Gravity | 1.011 | Normal |
| URINE SPOT PROTEIN | Nil | Normal |
| Glucose in Urine | Nil | Normal |
| Ketone | Nil | Normal |
| Bilirubin | Nil | Normal |
| Nitrite | Nil | Normal |
| Leukocytes | Nil | Normal |
| Blood in Urine | Nil | Normal |
| Pus Cells | Nil | Normal |
| RBC | Nil | Normal |
| Casts | Nil | Normal |
| Crystals | Nil | Normal |
| Epithelial Cells | Nil | Normal |
| **Vitamin D** | **12.32** | **At Risk (< 20)** |
| **VITAMIN B12** | **329** | Healthy (≥ 300) |
| PSA | 0.46 | Normal (< 4.0) |

---

### Employee 3 — BHADRESH SHAH · High Risk

*(Source: MRN 2045200470 from CSV, Dept + Location + HbA1c + BMI added, Uric Acid adjusted)*

| Field | Value | Status |
|---|---|---|
| **MRN** | 2045200470 | — |
| **Patient Name** | Mr. BHADRESH SHAH | — |
| **Age** | 58 Yrs | — |
| **Gender** | Male | — |
| **Dept** | Operations | — |
| **Location** | Pune | — |
| **BMI** | 29.2 | Obese Class I |
| **HbA1c** | 6.8% | **At Risk (≥ 6.5, diabetic range)** |
| Hemoglobin | 12.9 | **FLAGGED** (M < 13.5) |
| Total RBC Count | 3.78 | Normal |
| PCV | 38.0 | Normal |
| MCV | 100.5 | Slightly high |
| MCH | 34.1 | Normal |
| MCHC | 34.0 | Normal |
| RDW | 14.3 | Normal |
| RBC Count | 0 | — |
| WBC Morphology | 7540 | Normal |
| Neutrophils | 72.9 | Normal |
| Lymphocytes | 15.6 | Low |
| Monocytes | 7.9 | Normal |
| Eosinophils | 3.1 | Normal |
| Basophils | 0.5 | Normal |
| Abs. Neutrophil Count | 5490 | Normal |
| Abs. Lymphocyte Count | 1180 | Normal |
| Abs. Monocyte Count | 600 | Normal |
| Abs. Eosinophil Count | 240 | Normal |
| Abs. Basophil Count | 30 | Normal |
| Platelet Count (PLT) | 287000 | Normal |
| MPV | 10.2 | Normal |
| ESR | 20 | Elevated |
| Fasting Blood Sugar (FBS) | 151.79 | **At Risk (> 126)** |
| Blood Sugar (Postprandial) | 206.28 | Elevated |
| Total Cholesterol | 194.1 | Normal |
| Triglycerides | 194.8 | **FLAGGED** (> 175) |
| Chol-VLDL | 39.0 | Elevated |
| Chol-LDL | 117.1 | **FLAGGED** (> 100) |
| Chol-HDL | 38.0 | Normal |
| TC : HDL ratio | 5.11 | Elevated |
| LDL : HDL Ratio | 3.08 | Normal |
| Lipid Profile-TOTAL LIPID | 737.8 | — |
| Serum Creatinine | 1.62 | **FLAGGED** (> 1.4) |
| BUN | 13.6 | Normal |
| Uric Acid | 7.5 | **FLAGGED** (M > 7.0) |
| SGPT (ALT) | 19 | Normal |
| SGOT (AST) | 12.31 | Normal |
| T3 | 1.06 | Normal |
| T4 | 8.5 | Normal |
| TSH | 1.44 | Normal |
| Colour | Yellow | — |
| Odour | Non Specific | — |
| Appearence | Clear | — |
| PH | 5.0 | Normal |
| Specific Gravity | 1.018 | Normal |
| URINE SPOT PROTEIN | Nil | Normal |
| Glucose in Urine | Nil | Normal |
| Ketone | Nil | Normal |
| Bilirubin | Nil | Normal |
| Nitrite | Nil | Normal |
| Leukocytes | Nil | Normal |
| Blood in Urine | Nil | Normal |
| Pus Cells | Nil | Normal |
| RBC | Nil | Normal |
| Casts | Nil | Normal |
| Crystals | Nil | Normal |
| Epithelial Cells | Nil | Normal |
| **Vitamin D** | **9.83** | **At Risk (< 20)** |
| **VITAMIN B12** | **210** | Monitor (200–299) |
| PSA | 0.66 | Normal (< 4.0) |

---

### Employee 4 — SUNITA MEHTA · Monitor

| Field | Value | Status |
|---|---|---|
| **MRN** | EMP004 | — |
| **Patient Name** | Ms. SUNITA MEHTA | — |
| **Age** | 33 Yrs | — |
| **Gender** | Female | — |
| **Dept** | HR & Legal | — |
| **Location** | Pune | — |
| **BMI** | 21.2 | Normal |
| **HbA1c** | 5.2% | Normal |
| Hemoglobin | 10.8 | **FLAGGED** (F < 12.0) |
| Total RBC Count | 3.9 | Normal |
| PCV | 34.0 | Low |
| MCV | 82.0 | Normal |
| MCH | 27.8 | Low |
| MCHC | 31.5 | Normal |
| RDW | 15.2 | Elevated |
| RBC Count | 0 | — |
| WBC Morphology | 7200 | Normal |
| Neutrophils | 62.0 | Normal |
| Lymphocytes | 28.5 | Normal |
| Monocytes | 6.2 | Normal |
| Eosinophils | 2.9 | Normal |
| Basophils | 0.4 | Normal |
| Abs. Neutrophil Count | 4464 | Normal |
| Abs. Lymphocyte Count | 2052 | Normal |
| Abs. Monocyte Count | 446 | Normal |
| Abs. Eosinophil Count | 209 | Normal |
| Abs. Basophil Count | 29 | Normal |
| Platelet Count (PLT) | 560000 | **FLAGGED** (> 450,000) |
| MPV | 8.8 | Normal |
| ESR | 28 | Elevated |
| Fasting Blood Sugar (FBS) | 88.0 | Normal |
| Blood Sugar (Postprandial) | 112.0 | Normal |
| Total Cholesterol | 185.0 | Normal |
| Triglycerides | 120.0 | Normal |
| Chol-VLDL | 24.0 | Normal |
| Chol-LDL | 95.0 | Normal |
| Chol-HDL | 48.0 | Normal |
| TC : HDL ratio | 3.85 | Normal |
| LDL : HDL Ratio | 1.98 | Normal |
| Lipid Profile-TOTAL LIPID | 522.0 | — |
| Serum Creatinine | 0.68 | Normal |
| BUN | 9.2 | Normal |
| Uric Acid | 4.4 | Normal (F < 6.0) |
| SGPT (ALT) | 22 | Normal |
| SGOT (AST) | 24 | Normal |
| T3 | 1.18 | Normal |
| T4 | 7.9 | Normal |
| TSH | 1.90 | Normal |
| Colour | Pale Yellow | — |
| Odour | Non Specific | — |
| Appearence | Clear | — |
| PH | 6.0 | Normal |
| Specific Gravity | 1.010 | Normal |
| URINE SPOT PROTEIN | Nil | Normal |
| Glucose in Urine | Nil | Normal |
| Ketone | Nil | Normal |
| Bilirubin | Nil | Normal |
| Nitrite | Nil | Normal |
| Leukocytes | Nil | Normal |
| Blood in Urine | Nil | Normal |
| Pus Cells | Nil | Normal |
| RBC | Nil | Normal |
| Casts | Nil | Normal |
| Crystals | Nil | Normal |
| Epithelial Cells | Nil | Normal |
| **Vitamin D** | **14.5** | **At Risk (< 20)** |
| **VITAMIN B12** | **252** | Monitor (200–299) |
| PSA | N/A | Female |

---

### Employee 5 — HETALKUMAR HINDOCHA · High Risk

*(Source: MRN 2045201508 from CSV, Dept + Location + HbA1c + BMI added)*

| Field | Value | Status |
|---|---|---|
| **MRN** | 2045201508 | — |
| **Patient Name** | Mr. HETALKUMAR HINDOCHA | — |
| **Age** | 43 Yrs | — |
| **Gender** | Male | — |
| **Dept** | Engineering | — |
| **Location** | Surat | — |
| **BMI** | 27.5 | Overweight |
| **HbA1c** | 6.2% | Monitor (5.7–6.4, pre-diabetic) |
| Hemoglobin | 15.7 | Normal (M ≥ 13.5) |
| Total RBC Count | 5.94 | Normal |
| PCV | 46.2 | Normal |
| MCV | 77.8 | Slightly low |
| MCH | 26.4 | Low |
| MCHC | 33.9 | Normal |
| RDW | 13.8 | Normal |
| RBC Count | 0 | — |
| WBC Morphology | 6300 | Normal |
| Neutrophils | 55.3 | Normal |
| Lymphocytes | 32.7 | Normal |
| Monocytes | 6.9 | Normal |
| Eosinophils | 5.0 | Normal |
| Basophils | 0.1 | Normal |
| Abs. Neutrophil Count | 3480 | Normal |
| Abs. Lymphocyte Count | 2060 | Normal |
| Abs. Monocyte Count | 440 | Normal |
| Abs. Eosinophil Count | 320 | Normal |
| Abs. Basophil Count | 10 | Normal |
| Platelet Count (PLT) | 238000 | Normal |
| MPV | 9.2 | Normal |
| ESR | 4 | Normal |
| Fasting Blood Sugar (FBS) | 155.0 | **At Risk (> 126)** |
| Blood Sugar (Postprandial) | 239.0 | Elevated |
| Total Cholesterol | 215.0 | Normal |
| Triglycerides | 279.0 | **FLAGGED** (> 175) |
| Chol-VLDL | 56.0 | Elevated |
| Chol-LDL | 122.0 | **FLAGGED** (> 100) |
| Chol-HDL | 37.0 | Normal |
| TC : HDL ratio | 5.81 | Elevated |
| LDL : HDL Ratio | 3.30 | Normal |
| Lipid Profile-TOTAL LIPID | 948.0 | — |
| Serum Creatinine | 1.04 | Normal |
| BUN | 8.9 | Normal |
| Uric Acid | 7.6 | **FLAGGED** (M > 7.0) |
| SGPT (ALT) | 43 | **FLAGGED** (> 40) |
| SGOT (AST) | 21 | Normal |
| T3 | 0.99 | Normal |
| T4 | 7.17 | Normal |
| TSH | 2.20 | Normal |
| Colour | Yellow | — |
| Odour | Non Specific | — |
| Appearence | Clear | — |
| PH | 5.0 | Normal |
| Specific Gravity | 1.017 | Normal |
| URINE SPOT PROTEIN | Nil | Normal |
| Glucose in Urine | Nil | Normal |
| Ketone | Nil | Normal |
| Bilirubin | Nil | Normal |
| Nitrite | Nil | Normal |
| Leukocytes | Nil | Normal |
| Blood in Urine | Nil | Normal |
| Pus Cells | Nil | Normal |
| RBC | Nil | Normal |
| Casts | Nil | Normal |
| Crystals | Nil | Normal |
| Epithelial Cells | Nil | Normal |
| **Vitamin D** | **16.3** | **At Risk (< 20)** |
| **VITAMIN B12** | **186** | **At Risk (< 200)** |
| PSA | 0.89 | Normal (< 4.0) |

---

## Part 3 — Full Computation Walkthrough

Using the 5 employees above. Constants: `TOTAL_HEADCOUNT = 6` (1 untested employee implied), `TOTAL_TESTED = 5`.

---

### Step 1 — Panel Status Per Employee

| Employee | Vitamin D | Lipid | CBC | Diabetes | B12 | Kidney | Thyroid | Liver |
|---|---|---|---|---|---|---|---|---|
| E1 Priya | Healthy | Healthy | Healthy | Healthy | Healthy | Healthy | Healthy | Healthy |
| E2 Jothish | **At Risk** | Monitor† | Healthy | Healthy | Healthy | Healthy | Healthy | Healthy |
| E3 Bhadresh | **At Risk** | **At Risk**‡ | Monitor§ | **At Risk** | Monitor | **At Risk**¶ | Healthy | Healthy |
| E4 Sunita | **At Risk** | Healthy | **At Risk**∥ | Healthy | Monitor | Healthy | Healthy | Healthy |
| E5 Hetalkumar | **At Risk** | **At Risk**# | Healthy | **At Risk** | **At Risk** | Monitor** | Healthy | Monitor†† |

† 1 lipid flag (LDL > 100 only) → Monitor
‡ 2 flags (LDL 117.1 > 100 + Triglycerides 194.8 > 175) → At Risk
§ 1 flag (Hb 12.9 < 13.5 male) → Monitor
¶ 2 flags (Creatinine 1.62 > 1.4 + Uric Acid 7.5 > 7.0) → At Risk
∥ 2 flags (Hb 10.8 < 12.0 female + Platelet 560,000 > 450,000) → At Risk
# 2 flags (LDL 122 > 100 + Triglycerides 279 > 175) → At Risk
** 1 flag (Uric Acid 7.6 > 7.0) → Monitor
†† 1 flag (SGPT 43 > 40, SGOT 21 normal) → Monitor (not At Risk — need both)

---

### Step 2 — Per-Employee Health Score

Formula: `score = VitD×0.15 + Lipid×0.20 + CBC×0.15 + Diabetes×0.15 + B12×0.10 + Kidney×0.10 + Thyroid×0.10 + Liver×0.05`
(At Risk = 0, Monitor = 50, Healthy = 100)

**E1 Priya:**
`100×0.15 + 100×0.20 + 100×0.15 + 100×0.15 + 100×0.10 + 100×0.10 + 100×0.10 + 100×0.05`
= 15 + 20 + 15 + 15 + 10 + 10 + 10 + 5 = **100.0**

**E2 Jothish:**
`0×0.15 + 50×0.20 + 100×0.15 + 100×0.15 + 100×0.10 + 100×0.10 + 100×0.10 + 100×0.05`
= 0 + 10 + 15 + 15 + 10 + 10 + 10 + 5 = **75.0**

**E3 Bhadresh:**
`0×0.15 + 0×0.20 + 50×0.15 + 0×0.15 + 50×0.10 + 0×0.10 + 100×0.10 + 100×0.05`
= 0 + 0 + 7.5 + 0 + 5 + 0 + 10 + 5 = **27.5**

**E4 Sunita:**
`0×0.15 + 100×0.20 + 0×0.15 + 100×0.15 + 50×0.10 + 100×0.10 + 100×0.10 + 100×0.05`
= 0 + 20 + 0 + 15 + 5 + 10 + 10 + 5 = **65.0**

**E5 Hetalkumar:**
`0×0.15 + 0×0.20 + 100×0.15 + 0×0.15 + 0×0.10 + 50×0.10 + 100×0.10 + 50×0.05`
= 0 + 0 + 15 + 0 + 0 + 5 + 10 + 2.5 = **32.5**

---

### Step 3 — Risk Tier Per Employee

Count of At Risk panels:

| Employee | At Risk Count | Tier |
|---|---|---|
| E1 Priya | 0 | **Low Risk** |
| E2 Jothish | 1 (Vitamin D) | **Monitor** |
| E3 Bhadresh | 4 (VitD, Lipid, Diabetes, Kidney) | **High Risk** |
| E4 Sunita | 2 (Vitamin D, CBC) | **Monitor** |
| E5 Hetalkumar | 4 (VitD, Lipid, Diabetes, B12) | **High Risk** |

---

## DASHBOARD (index.html) — Computed Values

---

### KPI 1 — Health Index

```
Org Score = AVG(100.0, 75.0, 27.5, 65.0, 32.5) = 300.0 / 5 = 60.0 → Grade B (60–74)
```

> Production value (449 employees): **78 / A–**

---

### KPI 2 — Evaluated

```
TOTAL_TESTED    = 5  (rows in CSV)
TOTAL_HEADCOUNT = 6  (external constant)
Participation % = 5 / 6 × 100 = 83.3% → 83%
Not Tested      = 6 − 5 = 1
Tested bar      = 83%
Not Tested bar  = 17%
```

> Production value: **82%, 449 tested, 99 not tested**

---

### KPI 3 — Largest Risk (Metabolic Health)

Metabolic flag = `Total Cholesterol > 250` OR `Triglycerides > 175` OR `Chol-LDL > 100` OR `FBS > 126` OR `HbA1c ≥ 6.5`

| Employee | Flags | Has metabolic flag? |
|---|---|---|
| E1 Priya | None | No |
| E2 Jothish | LDL 150.6 > 100 | **Yes** |
| E3 Bhadresh | Trigly 194.8, LDL 117.1, FBS 151.79, HbA1c 6.8 | **Yes** |
| E4 Sunita | None | No |
| E5 Hetalkumar | Trigly 279, LDL 122, FBS 155 | **Yes** |

```
Metabolic flag count = 3 out of 5
Metabolic % = 3 / 5 × 100 = 60%
```

By dept:
```
Operations (E2, E3): both flagged → 2/2 = 100%
Engineering (E1, E5): E5 flagged → 1/2 = 50%
HR & Legal (E4): 0/1 = 0%
```

> Production values: **40% overall, OPS 18%, SALES 12%, ENG 10%**

---

### KPI 4 — Priority Action (Lipid Profile)

Lipid flag = `Total Cholesterol > 250` OR `Triglycerides > 175` OR `Chol-LDL > 100` OR `Chol-HDL < 30`

| Employee | Flags |
|---|---|
| E1 Priya | 0 |
| E2 Jothish | LDL 150.6 (1 flag) |
| E3 Bhadresh | LDL 117.1 + Trigly 194.8 (2 flags) |
| E4 Sunita | 0 |
| E5 Hetalkumar | LDL 122 + Trigly 279 (2 flags) |

```
Employees with any lipid flag = E2, E3, E5 = 3 out of 5
Lipid % = 3 / 5 × 100 = 60%
```

> Production value: **28%**

---

### Department Health Ranking

```
Engineering (E1=100, E5=32.5) → AVG = 66.25  → Rank #1
HR & Legal  (E4=65)            → AVG = 65.00  → Rank #2
Operations  (E2=75, E3=27.5)  → AVG = 51.25  → Rank #3
```

> Production values: **Marketing 84, Eng 80, Sales 76, Ops 68**

---

### Aggregated Diagnostic Profiles — pct per parameter

`pct = COUNT(flag) / TOTAL_HEADCOUNT × 100` (base = 5 employees)

| Parameter | Flag Definition | Flagged Employees | pct | Status |
|---|---|---|---|---|
| Vitamin D | VitD < 20 | E2, E3, E4, E5 | **4/5 = 80%** | At Risk |
| Lipid Profile | any lipid flag | E2, E3, E5 | **3/5 = 60%** | At Risk |
| CBC | any CBC flag | E3 (Hb), E4 (Hb + Plt) | **2/5 = 40%** | At Risk |
| Diabetes (HbA1c) | HbA1c ≥ 5.7 OR FBS > 110 | E3, E5 | **2/5 = 40%** | At Risk |
| Vitamin B12 | B12 < 300 | E3 (210), E4 (252), E5 (186) | **3/5 = 60%** | At Risk |
| Kidney | any kidney flag | E3 (Creat+UricAcid), E5 (UricAcid) | **2/5 = 40%** | At Risk |
| Thyroid | TSH outside 0.35–5.6 | None | **0/5 = 0%** | Healthy |
| Liver | SGPT > 40 OR SGOT > 50 | E5 (SGPT 43) | **1/5 = 20%** | Monitor |

> Production values: **VitD 42%, Lipid 28%, CBC 21%, Diabetes 12%, B12 15%, Kidney 3%, Thyroid 4%, Liver 5%**

---

### Workforce Risk Profiling — Right Panel

```
Low Risk   = 1/5 = 20%  (~1 employee)  — E1 Priya
Monitor    = 2/5 = 40%  (~2 employees) — E2 Jothish, E4 Sunita
High Risk  = 2/5 = 40%  (~2 employees) — E3 Bhadresh, E5 Hetalkumar
```

Breakdowns within each tier:

**Low Risk:**
```
Engineering Low % = 1 Low in Eng / 2 Eng total = 50%   (E1 is Low, E5 is High)
Age < 35 Low %    = 1 Low aged <35 / 1 aged <35 = 100% (only E1 is <35, she's Low Risk)
Female Low %      = 1 Low female / 2 females    = 50%  (E1 Low, E4 Monitor)
```

**Monitor:**
```
Operations Monitor % = 1 Monitor in Ops / 2 Ops total = 50%  (E2 Monitor, E3 High)
Age 35–45 Monitor %  = 0 Monitor aged 35-45 / 1 aged 35-45 = 0%  (E5 age 43 is High Risk)
Location Pune Monitor % = 1 Monitor in Pune / 2 in Pune = 50%  (E3 High, E4 Monitor)
```

**High Risk:**
```
Operations High % = 1 High in Ops / 2 Ops total = 50%   (E3 High, E2 Monitor)
Age > 45 High %   = 1 High aged >45 / 2 aged >45 = 50%  (E2 age 49 Monitor, E3 age 58 High)
Location Pune High % = 1 High in Pune / 2 in Pune = 50%
```

> Production values: **Low: Eng 71%, Age<35 68%, Female 65% · Monitor: Sales 32%, Age 35-45 29%, Hyderabad 28% · High: Ops 23%, Age>45 21%, Pune 18%**

---

## AI INSIGHTS (ai_insights.html) — Computed Values

---

### Priority Risk Radar — Gauge Values

**Vitamin D prevalence** (< 30 ng/mL, broad threshold):
```
E1: VitD 34 → normal
E2: 12.32 < 30 → Yes
E3: 9.83 < 30  → Yes
E4: 14.5 < 30  → Yes
E5: 16.3 < 30  → Yes
Prevalence = 4/5 = 80%
Employee count ≈ 80% × 5 = 4 employees
```

**Lipid at risk** (broad flag: T.Chol > 200 OR TGL > 150 OR LDL > 100 OR HDL < 40):
```
E2: LDL 150.6 > 100 → Yes
E3: LDL 117.1, TGL 194.8 → Yes
E5: LDL 122, TGL 279 → Yes
= 3/5 = 60%
```

**B12 below optimal** (< 300):
```
E3: 210 → Yes
E4: 252 → Yes
E5: 186 → Yes
= 3/5 = 60%
```

---

### Clinical Tier Profiles — Avg Values per Tier

**Low Risk** (E1 only):
```
Avg Vitamin D     = 34.0 ng/mL
Avg Chol-LDL      = 85 mg/dL
Avg HbA1c         = 5.0%
Avg VITAMIN B12   = 412 pg/mL
Avg Hemoglobin    = 13.5 g/dL
→ All markers: Normal
```

**Monitor** (E2, E4):
```
Avg Vitamin D     = (12.32 + 14.5) / 2  = 13.4 ng/mL  → At Risk
Avg Chol-LDL      = (150.6 + 95) / 2    = 122.8 mg/dL  → Borderline
Avg HbA1c         = (5.4 + 5.2) / 2     = 5.3%         → Normal
Avg VITAMIN B12   = (329 + 252) / 2     = 290.5 pg/mL  → Borderline
Avg Hemoglobin    = (14.9 + 10.8) / 2   = 12.85 g/dL   → Borderline (for respective genders)
```

**High Risk** (E3, E5):
```
Avg Vitamin D     = (9.83 + 16.3) / 2   = 13.1 ng/mL  → At Risk
Avg Chol-LDL      = (117.1 + 122) / 2   = 119.6 mg/dL → Abnormal
Avg HbA1c         = (6.8 + 6.2) / 2     = 6.5%         → Abnormal
Avg VITAMIN B12   = (210 + 186) / 2     = 198 pg/mL    → At Risk
Avg Hemoglobin    = (12.9 + 15.7) / 2   = 14.3 g/dL   → Normal avg (gender effects cancel)
```

---

## WORKFORCE (workforce.html) — Computed Values

---

### KPI Overview

```
Low Risk   62% → using 449 tested base = 278 employees
Monitor    26%                          = 117 employees
High Risk  12%                          =  54 employees
Wellness Score = 78 (org avg)
```

From 5 employees:
```
Low    = 1/5 = 20%  (E1)
Monitor = 2/5 = 40%  (E2, E4)
High   = 2/5 = 40%  (E3, E5)
Wellness Score = 60.0
```

---

### Parameter Risk Intelligence — By Department

**Vitamin D (< 20) by dept:**
```
Engineering (E1, E5): E5 flagged → 1/2 = 50%
Operations  (E2, E3): E2, E3 both flagged → 2/2 = 100%
HR & Legal  (E4):     E4 flagged → 1/1 = 100%
```

**Lipid (any flag) by dept:**
```
Engineering (E1, E5): E5 flagged → 1/2 = 50%
Operations  (E2, E3): both flagged → 2/2 = 100%
HR & Legal  (E4):     none → 0/1 = 0%
```

**HbA1c (≥ 5.7) by dept:**
```
Engineering (E1, E5): E5 (6.2) → 1/2 = 50%
Operations  (E2, E3): E3 (6.8) → 1/2 = 50%
HR & Legal  (E4):     none → 0%
```

---

### Parameter Risk Intelligence — By Gender

| Parameter | Female (E1, E4) | Male (E2, E3, E5) |
|---|---|---|
| Vitamin D (< 20) | E4 flagged → **1/2 = 50%** | E2, E3, E5 → **3/3 = 100%** |
| Lipid (any flag) | None → **0/2 = 0%** | E2, E3, E5 → **3/3 = 100%** |
| CBC (any flag) | E4 → **1/2 = 50%** | E3 → **1/3 = 33%** |
| B12 (< 300) | E4 (252) → **1/2 = 50%** | E3 (210), E5 (186) → **2/3 = 67%** |
| HbA1c (≥ 5.7) | None → **0/2 = 0%** | E3 (6.8), E5 (6.2) → **2/3 = 67%** |

Biggest gender gap: **Lipid** (M 100% vs F 0% = 100pp gap in this 5-person set)

> Production value biggest gap: **CBC / Anaemia (F 34% vs M 11%)**

---

### Parameter Risk Intelligence — Co-occurrence Matrix

`cell(A, B) = COUNT(flag A AND flag B) / COUNT(flag A) × 100`

Flagged populations:
- VitD: E2, E3, E4, E5 (4 employees)
- Lipid: E2, E3, E5 (3 employees)
- CBC: E3, E4 (2 employees)
- B12: E3, E4, E5 (3 employees)
- HbA1c: E3, E5 (2 employees)

| | VitD | Lipid | CBC | B12 | HbA1c |
|---|---|---|---|---|---|
| **VitD** | — | E2,E3,E5 in both → **3/4 = 75%** | E3,E4 → **2/4 = 50%** | E3,E4,E5 → **3/4 = 75%** | E3,E5 → **2/4 = 50%** |
| **Lipid** | E2,E3,E5 all VitD flagged → **3/3 = 100%** | — | E3 → **1/3 = 33%** | E3,E5 → **2/3 = 67%** | E3,E5 → **2/3 = 67%** |
| **CBC** | E3,E4 both VitD flagged → **2/2 = 100%** | E3 → **1/2 = 50%** | — | E3,E4 → **2/2 = 100%** | E3 → **1/2 = 50%** |
| **B12** | E3,E4,E5 all VitD flagged → **3/3 = 100%** | E3,E5 → **2/3 = 67%** | E3,E4 → **2/3 = 67%** | — | E3,E5 → **2/3 = 67%** |
| **HbA1c** | E3,E5 both VitD flagged → **2/2 = 100%** | E3,E5 both Lipid flagged → **2/2 = 100%** | E3 → **1/2 = 50%** | E3,E5 → **2/2 = 100%** | — |

> Production values: VitD↔Lipid 41%, Lipid↔HbA1c 38% (strongest pairs)

---

### High-Risk Cohort Explorer

| Cohort | Headcount | High-Risk count | % At Risk | Primary Risk Driver |
|---|---|---|---|---|
| Operations | 2 (E2, E3) | 1 (E3) | **50%** | Lipid + Diabetes |
| Engineering | 2 (E1, E5) | 1 (E5) | **50%** | Vitamin D + Lipid |
| HR & Legal | 1 (E4) | 0 | **0%** | CBC (Monitor only) |
| Age > 45 | 2 (E2 age 49, E3 age 58) | 1 (E3) | **50%** | HbA1c |
| Location Pune | 2 (E3, E4) | 1 (E3) | **50%** | Multi-factor |
| Location Surat | 2 (E2, E5) | 1 (E5) | **50%** | Vitamin D + Lipid |

Summary cards:
```
Total High-Risk  = 2 employees  (E3, E5)
Highest Dept %   = 50% (Operations or Engineering tied)
Lowest Dept %    = 0%  (HR & Legal)
Active Alerts    = 2   (Operations + Engineering — both have High Risk employees)
```

> Production values: **Total 54, Highest 23% (Ops), Lowest 5% (HR&Legal), Alerts 3**

---

## Notes for Developers

1. All dashboard pcts use `TOTAL_HEADCOUNT` (548) as denominator for clinical parameter prevalence and `TOTAL_TESTED` (449) as denominator for risk tier percentages.

2. The `Age` field in the CSV is a string like "49 Yrs". Parse the numeric part before applying age-group bucketing (<35 / 35-45 / >45).

3. `Gender` controls two flag thresholds: Hemoglobin (<13.5 Male, <12.0 Female) and Uric Acid (>7.0 Male, >6.0 Female).

4. `PSA` field is male-only. Skip or treat as N/A for female employees.

5. B12 values in some CSV rows appear as "< 148" (below detection limit). Treat these as 147 for computation purposes (clearly At Risk).

6. All panel weights must sum to 1.0: 0.15 + 0.20 + 0.15 + 0.15 + 0.10 + 0.10 + 0.10 + 0.05 = **1.00** ✓

7. The `Dept` column in the current CSV contains the placeholder value "Dept" for all rows. This must be replaced with actual department names before any dept-level computation.
