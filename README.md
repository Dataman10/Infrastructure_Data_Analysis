# Infrastructure Data Analysis – Canadian Projects

## 1. Project Overview
This repository contains a **comprehensive workflow for cleaning, filtering, enriching, and analyzing** a dataset of real-world project records. The dataset contains projects from multiple sources (DODGE, Infrastructure Canada, EU portals, US Grants, TxDOT, etc.) and includes both infrastructure and non-infrastructure records.

**Objective:**  
- Identify true infrastructure projects.  
- Flag megaprojects.  
- Normalize budgets and timestamps.  
- Enrich missing information using web sources and optional GenAI assistance.  
- Analyze failure rates and delivery/procurement methods.  

---

## 2. Dataset Overview
The raw CSV dataset contains the following key fields:  

| Field | Description |
|-------|------------|
| `title` | Short project/program title |
| `description` | Free-text project description |
| `url` | Link to source page |
| `country_name`, `state_name`, `city_name`, `county_name` | Location fields |
| `currency` | Project currency code |
| `budget` | Numeric raw budget from source |
| `ml.budget_ccy_bot.results.budget_usd` | Model-derived budget in USD (if available) |
| `budget_label` | Budget label (e.g., "Estimated Construction Cost") |
| `ml.status_simplified.results.identified_status` | Simplified project status |
| `ml.procurement_method_redo2.results.method` | Delivery/procurement method |
| `timestamp`, `timestamp_label`, `timestamps` | Project date(s) information |
| `sector / subsector` | Sector labels if available |
| `ml.sector_tags_method.results.level1_sector` | Higher-level sector tag |
| `ml.entity_enrich_redo1.results.*` | Entity type indicators |

**Challenges:**  
- Dataset contains both infrastructure and non-infrastructure projects.  
- Budget and timestamp fields are noisy and inconsistent.  
- Missing attributes exist for many projects.  

---

## 3. System Design & Workflow

### Step 1 – Setup
- Created working folder in my local system.  
- All outputs (code, datasets, PDF report) stored in this folder.  

### Step 2 – Load & Inspect
- Loaded CSV in Python using **pandas**.  
- Inspected: column names, sample rows, data types, missing values.  
- Identified obvious non-infrastructure projects (e.g., scholarships, research grants).  

### Step 3 – Data Cleaning
- Removed exact duplicates and rows marked as “DUPLICATE REPORT”.  
- Standardized column names and cleaned project titles.  
- Parsed JSON-like timestamp fields to usable Python dates.  
- Defined primary timestamp selection rule:  
  1. `project_start_date`  
  2. `estimated_start_date`  
  3. `publish_date`  

### Step 4 – Infrastructure Filtering
- Created `is_infrastructure_project` boolean column.  
- TRUE for physical asset projects: Transport, Water/Environment, Energy, Social Infrastructure, Industrial/Data.  
- FALSE for non-physical projects: scholarships, corporate revenue, training programs, general grants, research-only projects.  
- Filtering used **keywords from description, sector, subsector, level1_sector, and source**.  

### Step 5 – Budget Normalization
- Created `budget_usd_clean` field.  
- Logic:  
  - Use `ml.budget_ccy_bot.results.budget_usd` if available.  
  - Else, if `currency == USD` and budget numeric → use `budget`.  
  - Else → set `NaN`.  

### Step 6 – Megaproject Flag
- Added `is_megaproject` boolean column.  
- Rule: `budget_usd_clean >= 500,000,000 → True`.  

### Step 7 – Enrichment (Web + Optional GenAI)
- For subset of projects (megaprojects + sample others), used:  
  - Web browsing: official portals, PDF reports  
  - Optional GenAI: translation, summarization, inferring sector or lifecycle phase  
- Filled missing fields: `project_type`, `sector_main`, estimated/actual start/end dates, `delivery_method`, contract award dates, risk/failure notes  

### Step 8 – Failure & Delivery Method Analysis
- Created `failed_or_problematic` flag: TRUE if `status` indicates Failure, Abandoned, Delayed, or Canceled.  
- Grouped projects by `delivery_method` (Design-Bid-Build, Design & Build, PPP/Concession, Other/Unknown).  
- Computed: number of projects, number and percentage failed, patterns by sector/region.  
- Visualized failure rates using bar charts and tables.  


## 4. Code & Reproducibility
- Code in `notebook.ipynb` includes all steps: loading CSV, cleaning, filtering, budget/timestamp normalization, megaproject flagging, dataset enrichment, failure/delivery analysis.  
- Written for **reproducibility and shareability**.  

---

## 5. Deliverables
| File | Description |
|------|-------------|
| `infra_projects_clean.csv` | Final cleaned and enriched infrastructure dataset |
| `notebook.ipynb` | Python code for full analysis workflow |
| `report.pdf` | 2–4 page report covering methodology, failure/delivery analysis, uncertainties |
| `README.md` | This file |

---

## 6. Tools & Technologies
- **Python:** pandas, numpy, matplotlib/seaborn  
- **IDE:** VS Code 
- **Other:** Web research, GenAI (optional enrichment)  

---

## 7. Key Outcomes
- Identified and cleaned true infrastructure projects.  
- Flagged megaprojects based on budget threshold.  
- Normalized budget and timestamp fields.  
- Conducted failure analysis by delivery/procurement method.  
- Generated insights for planning, risk assessment, and reporting.  

---

## 8. Notes & Limitations
- AI assistance used **only for summarization and inference**, not report writing.  
- Dataset may have missing attributes; enrichment done for priority projects.  
- Results may be biased by source coverage or incomplete information.  
- Recommendations include expanding sources, improving labeling, and filling missing budget/timestamp fields.  

---

## 9. Future Work
- Extend enrichment to all projects.  
- Incorporate additional datasets to validate megaproject thresholds.  
- Perform predictive analytics on project success/failure.  
- Add interactive dashboards for visualization.  

---


