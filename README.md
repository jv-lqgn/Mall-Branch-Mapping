# 🏬 Mall Branch Mapping

> Pipeline to map SM Store branch names from the 3-Day Sale Schedule to their corresponding `branch_id` in the `dim_mall` dimension table.

---

## 📌 Overview

The **Mall Branch Mapping** pipeline resolves inconsistent branch naming conventions across two datasets:

- **3-Day Sale Schedule** (`YEAR_2026_3-Day_Sale_Schedule.xlsx`) — contains branch names as entered by the business team (e.g., `"North EDSA"`, `"CDO Uptown"`, `"Center - Imus"`)
- **dim_mall** (`dim_mall_0415.csv`) — the master mall dimension table with canonical `branch_name` and `branch_id` (e.g., `"SM Store North Edsa"`)

The pipeline standardizes naming, applies a curated manual mapping dictionary, and outputs an enriched CSV with `branch_id` resolved for downstream analytics.

---

## 📁 Repository Structure

```
Mall-Branch-Mapping/
├── brand_to_mall_mapping.ipynb     # Main pipeline notebook
├── dim_mall_0415.csv               # Master mall dimension table (80 branches)
├── YEAR_2026_3-Day_Sale_Schedule.xlsx  # Input: 2026 3-Day Sale Schedule
├── 3ds_schedule_mapped.csv         # Output: enriched schedule with branch_id
└── README.md
```

---

## ⚙️ Pipeline Steps

| Step | Description |
|------|-------------|
| 1 | Load `dim_mall` CSV and sale schedule XLSX into pandas DataFrames |
| 2 | Normalize branch names — strip `"SM Store "` prefix and whitespace |
| 3 | First-pass merge on cleaned branch name |
| 4 | Identify unmatched rows (`branch_id = NaN`) |
| 5 | Apply `MANUAL_MAP` dictionary for alias names, casing, and SM Center variants |
| 6 | Re-merge using mapped names to resolve remaining `branch_id`s |
| 7 | Validate match rate and flag unresolvable rows |
| 8 | Export final output to `3ds_schedule_mapped.csv` |

---

## 📊 Output Schema

| Column | Type | Description |
|--------|------|-------------|
| `branch_id` | Integer | Resolved branch ID from `dim_mall`. Null if unmatched. |
| `branch_name` | String | Full SM Store branch name from `dim_mall` |
| `branch_name_mapped` | String | Normalized name used for the join |
| `year` | Integer | Sale year |
| `period` | String | Half-year period: `H1` or `H2` |
| `sale_date` | Date | Individual sale day date |
| `access_type` | String | `early access` or `regular` |
| `sale_duration` | String | Duration code: `3ds`, `4ds`, or `5ds` |

---

## 📈 Match Rate (2026)

| Metric | Count | % |
|--------|-------|---|
| Total rows | 754 | — |
| Matched (`branch_id` resolved) | 638 | 84.6% |
| Unmatched (`branch_id = NaN`) | 116 | 15.4% |

Unmatched rows correspond to **SM Center branches** not yet in `dim_mall`, **chain-wide promos** (`CHAIN PROMO`), and **specialty malls** (The Podium, S'Maison). These are expected and documented in the notebook appendix.

---

## 🚀 How to Run

1. Clone this repository
2. Place the latest `dim_mall` CSV and sale schedule XLSX in the repo root (or update the path variables in Step 1 of the notebook)
3. Open `brand_to_mall_mapping.ipynb` in Jupyter or Databricks
4. Run all cells sequentially
5. Review the Step 7 validation output and check the match rate
6. The output CSV will be written to the path defined in `OUTPUT_PATH`

### Requirements

```
pandas
openpyxl
```

Install with:

```bash
pip install pandas openpyxl
```

---

## ⚠️ Known Unresolved Branches

The branches below have no matching entry in `dim_mall` and will retain `branch_id = NaN` in the output. Coordinate with the data team to add them to the dimension table or confirm exclusion.

| Sale Schedule Name | Notes |
|--------------------|-------|
| Center - Angono | SM Center variant; not in dim_mall |
| Center - Lemery | SM Center variant; not in dim_mall |
| Center - Muntinlupa | SM Center variant; not in dim_mall |
| Center - Ormoc | SM Center variant; not in dim_mall |
| Center - Pulilan / SM Center Pulilan | Duplicate naming; not in dim_mall |
| Center - Sagandaan / SM Center Sangandaan | Duplicate naming; not in dim_mall |
| Center - Tuguegarao Downtown / SM Center Tuguegarao Downtown | Duplicate naming; not in dim_mall |
| Center Shaw / SM Center Shaw | Duplicate naming; not in dim_mall |
| Center – Las Piñas / SM Center Las Piñas | Duplicate naming; not in dim_mall |
| Center – San Pedro / SM Center San Pedro | Duplicate naming; not in dim_mall |
| SM Center Dagupan | Not in dim_mall |
| S'Maison | Specialty mall; not in dim_mall |
| The Podium | Specialty mall; not in dim_mall |
| CHAIN PROMO | Chain-wide promo — not a branch; flagged via `is_chain_promo = True` |

The following entries are mapped but **require business confirmation** before use in production:

| Sale Schedule Name | Mapped To | Confidence |
|--------------------|-----------|------------|
| Center - Imus / SM Center Imus | Bacoor | Medium — verify location |
| Center - Pasig / SM Center Pasig | East Ortigas | Medium — verify location |

---

## 📝 How to Add a New Branch Mapping

When a new sale schedule introduces an unrecognized branch name:

1. Identify the unmatched name in the Step 4 notebook output
2. Find the correct `dim_mall` entry by checking `branch_name_clean` (after stripping `"SM Store "`)
3. Add the entry to `MANUAL_MAP` in Step 5:
   ```python
   'Sale Schedule Name': 'dim_mall Clean Name'
   ```
4. If no `dim_mall` entry exists, set the value to `None` — the row will retain `branch_id = NaN`
5. Re-run Steps 5–8 to apply the new mapping

---

## 📄 Documentation

Full user guide: [`Mall_Branch_Mapping_User_Guide.docx`](documents/Mall_Branch_Mapping_User_Guide.docx)

---

*Digital Advantage Corporation – Analytics and Insights*
