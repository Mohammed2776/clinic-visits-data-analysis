# MediCare Outpatient Visits — Mini-Project 2

**Python for Full Stack Data Science with AI & Generative AI · Naresh IT**  
Lead Trainer: Ajit Byru

## Business question
Use MediCare's 2025 outpatient visit records to identify workload and waiting-time pressure, understand revenue by department, describe age/consultation patterns, and recommend a practical 2026 action.

## Files
- `clinic_visits_2025.csv` — original raw export (2,423 rows, 13 columns).
- `clinic_visits_analysis_completed.ipynb` — completed and executed notebook, including code, charts, readings, and recommendation.
- `clinic_visits_clean.csv` — analysis-ready data (2,397 rows, 16 columns).
- `MediCare_Mini_Project_2_Slides.pptx` — five-slide summary.
- `MediCare_Two_Minute_Pitch.txt` — speaking notes.

## Cleaning log summary
The original export contains 2,423 rows and 13 columns. Exact duplicate removal leaves 2,400 rows. Three records have impossible ages and are rejected, leaving **2,397 rows**. Five negative waiting-time values are corrected by absolute value. Dates are parsed by explicit format, departments are standardised, 118 missing fees are imputed with department medians, and 13 ₹50,000 fees are retained as procedure-like charges rather than deleted as outliers.

| # | Problem | Rows affected | Fix | Why this fix |
|---:|---|---:|---|---|
| 1 | Exact duplicate visit rows | 23 duplicate rows | Drop exact duplicate rows first | Exact matches add no new information; this avoids double-counting without discarding distinct visits. |
| 2 | Department labels have inconsistent case/whitespace | 60 rows differ from strip/title standardisation | Strip whitespace, title-case, then preserve the acronym ENT | This changes formatting only and consolidates six real departments; preserving ENT avoids an awkward “Ent” label. |
| 3 | Mixed date formats in visit_date | 2400 unique rows; 0 ISO, 0 DD/MM/YYYY, 0 DD-Mon-YYYY strings | Parse each format explicitly with errors=coerce and fill missing parses | Prevents ambiguous day/month interpretation and makes unparsed dates visible. |
| 4 | Impossible ages (age ≤ 0 or > 110) | 3 rows: V101228 (0), V102161 (412), V102242 (199) | Reject the records and retain the visit IDs in the audit trail | The true age cannot be recovered from other columns; guessing would fabricate data. |
| 5 | Negative wait time sign errors | 5 rows | Replace negative values with absolute values | The project brief identifies these as sign errors; magnitude is preserved, but source logs should confirm the assumption. |
| 6 | IQR upper-fence fee outliers | 13 rows above ₹1,735.00; each is ₹50,000 in Cardiology or Orthopedics | Retain and mark is_procedure=True for fee == 50000 | Repeated exact values in relevant departments look like a distinct procedure charge, not random errors; confirm with billing codes. |
| 7 | Missing fee values | 118 rows after duplicate removal | Impute department-specific median | Preserves department-level fee differences and is robust to extreme procedure charges. |


### Important assumptions
- `is_procedure` is a **provisional analytical flag** based on the repeated ₹50,000 charge in Cardiology and Orthopedics. There is no explicit procedure column in the supplied CSV; billing codes must confirm this before operational reporting.
- The project brief mentions a first parse yielding 83 Sunday visits. In the pandas runtime used to execute this notebook, the mixed parser returned zero Sundays. Because parser behaviour can vary by pandas version and the clinic is closed Sundays, the notebook still demonstrates the safer explicit-format parser and verifies the final weekday distribution. The supplied data parses to zero Sunday visits in this run.
- Department medians are calculated from observed fees after duplicate removal and before imputation. The three invalid-age visits are excluded from the analysis-ready dataset.

## Findings (from the supplied data)
1. **Workload:** General Medicine is busiest with 790 visits; Monday is busiest with 495 visits.
2. **Waiting:** General Medicine has the highest 90th-percentile wait (49.0 minutes; mean 37.8 minutes).
3. **Fees:** Cardiology mean fee is ₹2,021, compared with median ₹1,260; the ₹50,000 procedure-like values pull the mean upward.
4. **Age and consultation length:** Pearson correlation is 0.767; the fitted line rises by 2.08 minutes per decade. This is association, not causation.
5. **Monthly revenue:** Highest recorded month is Jan (₹362,130); lowest is Jul (₹145,750). Total recorded revenue after imputation is ₹2,554,950.
6. **Own question — revenue concentration:** the top 10% of visits by fee (240 visits) account for ₹955,400, or **37.4%** of recorded revenue. Cardiology and Orthopedics together account for **59.3%**. This statistic is sensitive to the provisional procedure flag and imputed fees.

## Recommendation
Administrator, the 2025 records show that General Medicine handled the most visits (790 of 2,397), so it should be the first place to review roster coverage and peak-hour queue management. Waiting-time comparisons show General Medicine has the highest 90th-percentile wait (49.0 minutes); review its appointment spacing, arrival surges and staff availability before making a permanent staffing change. The IQR rule flagged 13 high fees, but each was ₹50,000 in Cardiology or Orthopedics, so these values were retained and separately flagged rather than deleted; confirm their meaning against billing/procedure codes. Missing fees were filled with department medians, and three impossible-age records were excluded. In 2026, trial a targeted queue review in the highest-p90 department and track median and 90th-percentile wait weekly alongside visits per staffed hour. Before treating revenue changes as growth, validate procedure coding and investigate the January revenue peak. With appointment timestamps, staffing rosters and verified billing codes, the clinic could distinguish capacity problems from case mix and one-off charges.

## Five interview questions — suggested answers

### 1. Why did you keep the ₹50,000 fees when the outlier rule flagged them?
The 1.5×IQR fence is a screening rule, not proof of error. All 13 flagged values are exactly ₹50,000 and occur in Cardiology or Orthopedics, which is consistent with a different procedure charge. I retained them and added a provisional `is_procedure` flag rather than capping or deleting potentially valid revenue. I would verify them against billing/procedure codes before publishing final financial reports.

### 2. Your first date parse produced 83 Sunday visits. How did you catch it, and how did you fix it?
I checked the weekday distribution and knew it contradicted the operating rule that the clinic is closed on Sundays. I then compared ISO-looking raw strings (`YYYY-MM-DD`) with the parsed output to detect day/month interpretation problems. I parsed ISO, slash-separated day-first, and month-name dates separately with explicit formats and `errors='coerce'`, combined the results, and asserted there were no unparsed dates and no Sunday visits. Note: this execution environment's mixed parser returned zero Sundays, but the explicit parser is safer across pandas versions.

### 3. Why per-department median for missing fees instead of the overall median or the mean?
Departments have different fee distributions, so the overall median could understate or overstate the expected fee for a particular department. The department median preserves those differences. Median is also more robust than mean when high procedure-like charges are present. This is still an imputation assumption and should be reviewed if missingness is related to billing practice.

### 4. Cardiology's mean fee is ₹2,021 and its median is ₹1,260. Which one goes in the annual report?
For the typical fee per visit, I would report the median ₹1,260 because the ₹50,000 charges pull the mean upward. I would report the mean as a separate financial metric and break out procedure revenue after confirming the billing codes. The two statistics answer different questions, so I would label them rather than silently choosing one.

### 5. What is one decision in your log you would change if you had more data?
I would replace the provisional procedure flag based on the ₹50,000 amount with a verified procedure/billing code. With itemised charges and billing definitions, I could distinguish a true procedure from a data-entry error without relying on a fee threshold.

## Reproducibility
Keep the CSV and notebook in the same directory. Install `numpy`, `pandas`, `matplotlib`, and Jupyter, then use **Restart Kernel and Run All**. No `sklearn` or predictive model is used.
