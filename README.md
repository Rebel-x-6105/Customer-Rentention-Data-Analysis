# Customer Retention & Churn Analysis

[![Python Version](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![Database](https://img.shields.io/badge/Database-SQLite-003B57.svg)](https://www.sqlite.org/)
[![Status](https://img.shields.io/badge/Status-Complete-success.svg)]()
[![Documentation](https://img.shields.io/badge/Executive%20Report-PDF-red.svg)](Customer_Retention_Analysis_Executive_Report.pdf)

An end-to-end data analytics and customer intelligence project examining subscriber churn, customer lifetime value (CLTV), customer service friction, and contract commitment dynamics for an OTT/subscription platform. 

This repository contains the complete analytical pipeline: SQL database extraction, data cleaning, feature engineering, exploratory data analysis (EDA), statistical correlation modeling, data visualization, and an executive-ready PDF report for business leadership.

---

## Table of Contents

- [Business Context & Problem Statement](#business-context--problem-statement)
- [Executive Summary & Core Metrics](#executive-summary--core-metrics)
- [Key Insights & Findings](#key-insights--findings)
- [Data Architecture & Pipeline](#data-architecture--pipeline)
- [Repository Structure](#repository-structure)
- [Visualizations & Executive Report](#visualizations--executive-report)
- [Strategic Recommendations & Roadmap](#strategic-recommendations--roadmap)
- [Getting Started & Usage](#getting-started--usage)
- [Technologies & Libraries](#technologies--libraries)

---

## Business Context & Problem Statement

In recurring-revenue subscription businesses (such as streaming platforms and SaaS), long-term profitability is governed by **Customer Retention** rather than customer acquisition alone. Acquiring a new customer costs **5 to 7 times more** than retaining an existing one. 

When subscribers churn:
1. **Immediate Revenue Leak:** Monthly Recurring Revenue (MRR) drops immediately.
2. **Loss of Customer Lifetime Value (CLTV):** Future renewal cycles, cross-sells, and margin expansion disappear.
3. **Marketing Inefficiency:** Customer acquisition cost (CAC) is never recovered if the customer churns before reaching payback tenure.

**Project Objectives:**
- Diagnose the root causes behind subscriber cancellations.
- Evaluate the vulnerability of subscription plan tiers, billing frequencies, and marketing channels.
- Quantify customer support friction as a leading indicator of churn.
- Provide leadership with actionable operational levers and a prioritized 30-60-90 day retention roadmap.

---

## Executive Summary & Core Metrics

Across the evaluated subscriber cohort, the key performance metrics are:

| Metric | Value | Business Interpretation |
| :--- | :---: | :--- |
| **Overall Churn Rate** | **28.6%** | 6 out of 21 customers canceled their subscription |
| **Portfolio Retention Rate** | **71.4%** | 15 out of 21 customers remain active |
| **Average Revenue Per User (ARPU)** | **\$18.85 / mo** | Average monthly recurring billing per subscriber |
| **Monthly Revenue at Risk (MRR Lost)** | **\$73.94 / mo** | 18.7% of total monthly portfolio revenue has been lost |
| **Annualized Revenue Leak** | **\$887.28 / yr** | Direct annual run-rate deficit from churned cohort |
| **Support Escalation Churn Rate** | **100.0%** | Every customer with an escalated complaint churned ($r = 0.77$) |
| **Average Active Customer Tenure** | **1,578 Days** | Retained subscribers demonstrate sustained long-term loyalty |
| **Average Churned Customer Tenure** | **285 Days** | Churned customers drop off rapidly within early contract cycles |

---

## Key Insights & Findings

### 1. Plan Tier Vulnerability
* **Basic Plan (60.0% Churn):** Severe drop-off (3 of 5 churned; avg. charge \$10.59/mo). Basic subscribers exhibit high price sensitivity, experience low switching friction, and feel a lack of premium features.
* **Standard Plan (22.2% Churn):** Moderate stability (2 of 9 churned; avg. charge \$13.77/mo).
* **Premium Plan (14.3% Churn):** Highest retention (6 of 7 retained; avg. charge \$31.28/mo). Premium customers perceive superior product value and demonstrate the strongest loyalty.

### 2. Contract Commitment as a Retention Shield
* **Month-to-Month Contracts:** **55.6% Churn** (5 of 9 churned). Monthly billing creates frequent decision cycles where users re-evaluate the service.
* **Annual Contracts:** **8.3% Churn** (only 1 of 12 churned). Annual commitments provide a **6.7x higher retention shield**.

### 3. Acquisition Channel Health & The Referral Leak
* **Organic Sign-ups:** **0.0% Churn** (9 of 9 retained). Strongest product-market fit and organic intent.
* **Paid Advertising:** **16.7% Churn** (5 of 6 retained). Healthy commercial acquisition channel.
* **Referral Sign-ups:** **83.3% Churn** (5 of 6 churned!). Referral bonuses attract incentive/trial hunters who cancel as soon as full-price billing commences. One customer explicitly noted *"Forgot to cancel trial"*.

### 4. Customer Support Escalations as the Primary Red Alert
* **Escalated Complaints (`escalations == 'Y'`):** **100.0% Churn** (4 of 4 churned).
* **Standard Complaints (`escalations == 'N'`):** **66.7% Churn** (2 of 3 churned).
* **No Complaints Logged:** **0.0% Churn** (14 of 14 retained).
* **Correlation:** **$r = 0.77$** between ticket escalation and cancellation. Support friction is the single strongest immediate predictor of churn.

### 5. Stated Cancellation Reasons
* **Switched to Competitor (33.3%):** Direct competitive alternative pressure.
* **Price / Perceived Value (16.7%):** Fee perceived as too expensive relative to utility.
* **Content Catalog Deficit (16.7%):** Exhausted content options or missing genres.
* **Streaming Quality Issues (16.7%):** Playback buffering and technical latency.
* **Forgot to Cancel Trial (16.7%):** Bonus hunter drop-off.

---

## Data Architecture & Pipeline

The project extracts and unifies data from an SQLite database (`customer_churn.db`):

```
                   +------------------------+
                   |   customer_churn.db    |
                   +-----------+------------+
                               |
         +---------------------+---------------------+
         |                     |                     |
         v                     v                     v
   +------------+       +---------------+       +------------+
   |db_customer |       |db_subscription|       | db_support |
   |  (21 rows) |       |   (21 rows)   |       |  (9 rows)  |
   +-----+------+       +-------+-------+       +-----+------+
         |                      |                     |
         |                      v                     |
         |         Feature Engineering:               |
         |         • Datetime parsing                 |
         |         • churn_flag (1 / 0)               |
         |         • tenure_days calculation          |
         |         • churn_risk tiering               |
         |                      |                     |
         v                      v                     v
+------------------+   +-------------------+  +------------------+
| Demographic Clean|   | Cleaned Contracts |  | Deduplicated Logs|
| • Gender norm    |   +---------+---------+  | • Latest ticket  |
| • State-country  |             |            | • Complaint count|
+--------+---------+             |            +--------+---------+
         |                       |                     |
         +-----------------------+---------------------+
                                 |
                                 v
                 +-------------------------------+
                 |    exported_churn_data.csv    |
                 | (21 rows x 21 analytical cols)|
                 +-------------------------------+
```

### Data Cleansing & Preparation Steps:
1. **Schema Inspection:** Used SQLite `PRAGMA table_info` and dynamic DataFrame ingestion in pandas.
2. **Demographic Standardization:** Standardized gender labels (`Men`/`Women` $\rightarrow$ `Male`/`Female`) and imputed missing country values using state-to-country dictionary mappings.
3. **Column Pruning:** Dropped sparse or uninformative columns (`interests`, `pincode`, `col_1`, `comment`).
4. **Timestamp Parsing:** Converted dates (`dob`, `subscription_start_date`, `renewal_date`, `cancellation_date`, `complaint_date`) to `datetime64[ns]`.
5. **Support Ticket Deduplication:** Resolved multi-ticket users by preserving the latest complaint date while engineering a cumulative `complaint_count` variable.
6. **Feature Engineering:**
   - `churn_flag`: Binary indicator (1 if `cancellation_date` is not null, 0 otherwise).
   - `tenure_days`: Active days between subscription start and cancellation (or current date).
   - `churn_risk`: Categorical tiering based on `churn_score` (`low` <50, `med` 50–69, `high` $\ge$70).
   - `escalations`: Binary encoded (Y = 1, N = 0).

---

## Repository Structure

```text
custemer retention analysis/
│
├── README.md                                     # Project documentation & reference
├── Customer_Retention_Analysis_Executive_Report.pdf # 4-page leadership executive report
│
├── churn_analysis.ipynb                          # Complete end-to-end analysis notebook (104 cells)
├── customer_churn.ipynb                          # Initial exploration & drafting notebook (49 cells)
│
├── customer_churn.db                             # Source SQLite database (3 relational tables)
├── test_database.sqlite                          # SQLite DDL/DML demonstration database
│
├── exported_churn_data.csv                       # Final cleaned & engineered analytical dataset
├── exported_customer_churn.csv                   # Preliminary export from baseline exploration
│
├── report_page_1.png                             # Executive Summary & Data Pipeline preview
├── report_page_2.png                             # Core Churn Drivers & Behavioral Analysis preview
├── report_page_3.png                             # Customer Experience & Financial Exposure preview
└── report_page_4.png                             # Strategic Recommendations & Action Plan preview
```

---

## Visualizations & Executive Report

The analysis is accompanied by a **4-Page Executive Report** formatted specifically for non-technical stakeholders and business leadership:

- **Page 1: Executive Summary, Business Context & Data Pipeline**  
  Executive scorecard, bottom-line KPIs, and non-technical explanation of the data cleansing pipeline.
- **Page 2: Core Churn Drivers & Behavioral Analysis**  
  Visual breakdown of churn by Plan Tier (Basic vs Premium), Contract Type (Monthly vs Annual), and Acquisition Channel (Organic vs Referral).
- **Page 3: Customer Experience, Root Causes & Financial Exposure**  
  Support escalation analysis ($r = 0.77$), exit survey reasons, financial impact table, and geographic performance.
- **Page 4: Strategic Recommendations & 30-60-90 Day Action Plan**  
  5 concrete operational initiatives, prioritized execution roadmap, and document governance.

You can view the full PDF here:  
👉 **[Download / View Customer_Retention_Analysis_Executive_Report.pdf](Customer_Retention_Analysis_Executive_Report.pdf)**

---

## Strategic Recommendations & Roadmap

### The Five Strategic Pillars

1. **Red-Flag Escalation Protocol (Customer Care)**
   - *Action:* Automated CRM trigger on escalated tickets routing directly to a Senior Concierge with a **<2 hour SLA** and authority to issue goodwill service credits.
   - *Expected Impact:* Recovers 40–50% of at-risk subscribers, protecting ~$30–$40/mo in recurring revenue.
2. **Annual Commitment Upgrade Campaign (Pricing & Packaging)**
   - *Action:* Launch an in-app "Switch to Annual, Get 2 Months Free" campaign targeted at month-to-month subscribers at Day 45 of tenure.
   - *Expected Impact:* Moves subscribers from the high-churn monthly tier (55.6%) into the low-churn annual tier (8.3%).
3. **Gated Referral Rewards (Growth Marketing)**
   - *Action:* Restructure referral incentives to require 15+ hours of streaming and completion of the first renewal cycle before releasing bonuses.
   - *Expected Impact:* Eliminates free-trial bonus hunters and cuts referral churn in half.
4. **Basic Plan Value Refresh (Product Strategy)**
   - *Action:* Upgrade Basic tier with 1080p Full HD resolution, mobile offline downloads, and an algorithmic onboarding guide during the first 14 days.
   - *Expected Impact:* Closes the competitive value gap and brings Basic churn down toward 30%.
5. **Automated Churn Risk Alert & Interventions (Retention Operations)**
   - *Action:* Trigger automated retention check-ins and personalized content recommendations whenever a user's Churn Risk Score exceeds 50.
   - *Expected Impact:* Intercepts dissatisfied users before they hit the cancellation button.

### Implementation Timeline

| Phase | Horizon | Key Operational Initiatives | Target Milestone & KPI |
| :--- | :---: | :--- | :--- |
| **Phase 1** | Days 1–30 | • Deploy 2-hour Red-Flag Escalation Protocol<br/>• Retool Referral Program with 60-day milestone<br/>• Triage active complaints in Karnataka & Meghalaya | • 0% Escalation Churn<br/>• Eliminate trial bonus abuse<br/>• Immediate account recovery |
| **Phase 2** | Days 31–60 | • Launch "Switch to Annual, Get 2 Months Free"<br/>• Deploy real-time Churn Risk alerts (score > 50)<br/>• Introduce exit survey counter-offers | • 25% Monthly-to-Annual shift<br/>• 100% intervention on at-risk accounts |
| **Phase 3** | Days 61–90 | • Overhaul Basic Plan (1080p + offline downloads)<br/>• Optimize regional CDN routing in high-buffering hubs<br/>• Refresh onboarding curation engine | • Basic churn cut to <35%<br/>• 40% reduction in streaming complaints |

---

## Getting Started & Usage

### Prerequisites
- Python 3.10+
- SQLite3
- Jupyter Notebook / JupyterLab or VS Code

### Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/customer-retention-analysis.git
   cd "customer-retention-analysis"
   ```

2. Create and activate a virtual environment:
   ```bash
   python -m venv venv
   # On Windows:
   venv\Scripts\activate
   # On macOS/Linux:
   source venv/bin/activate
   ```

3. Install required packages:
   ```bash
   pip install pandas numpy matplotlib seaborn jupyter
   ```

### Running the Analysis
1. Launch Jupyter Notebook:
   ```bash
   jupyter notebook
   ```
2. Open [`churn_analysis.ipynb`](churn_analysis.ipynb) to view and execute the full analysis pipeline.
3. To inspect the exported dataset directly, load [`exported_churn_data.csv`](exported_churn_data.csv) in Python, Excel, or BI tools (Power BI / Tableau).

---

## Technologies & Libraries

- **Language:** Python 3.12
- **Database:** SQLite3
- **Data Manipulation:** `pandas`, `numpy`
- **Data Visualization:** `matplotlib`, `seaborn`
- **Document Generation:** `matplotlib.backends.backend_pdf` (PdfPages)
- **Environment:** Jupyter Notebook, VS Code

---

## Author & Governance

- **Project:** Customer Retention & Churn Diagnostic
- **Document Version:** 1.0
- **Distribution:** Executive Leadership, Product Engineering, Growth Marketing, Customer Care
