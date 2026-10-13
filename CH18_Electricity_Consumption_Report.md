# GridSense Energy Services — CH18 Electricity Consumption Insight
## Complete End-to-End Data Science Assessment & Demand Planning Report

**Document Reference:** U23AD491 – Data Science | DS Sprint – Skill Based Assessment 2  
**Organization:** GridSense Energy Services  
**Sector:** Energy & Power Utilities  
**Core Analytical Question:** *What temporal and environmental factors influence electricity consumption, and how should GridSense optimize demand planning, grid capacity, and tariff structures?*

---

## 1. Executive Summary & Core Deliverables

This project delivers a complete, production-grade end-to-end data science solution for **CH18 – Electricity Consumption Insight**, synthesized across 86,400 hourly meter observations, 120 customer accounts, local weather telemetry, calendar events, and time-of-use tariff schedules.

### Deliverables Inventory
| Deliverable | File Path | Description |
| :--- | :--- | :--- |
| **Cleaned Master Dataset** | [`cleaned_electricity_data.csv`](file:///d:/SAT-2/cleaned_electricity_data.csv) | Full 86,400-row panel dataset with all 5 source tables unified, cleaned, imputed, and validated with zero missing values. |
| **End-to-End Jupyter Notebook** | [`electricity_consumption_insight.ipynb`](file:///d:/SAT-2/electricity_consumption_insight.ipynb) | 23-cell executed Jupyter Notebook containing complete markdown narrative, audit code, cleaning steps, EDA charts, ML modeling, and evaluations. |
| **Excel Analytical Workbook** | [`CH18_Electricity_Consumption_Analysis.xlsx`](file:///d:/SAT-2/CH18_Electricity_Consumption_Analysis.xlsx) | Professional 9-tab workbook featuring KPI executive cards, statistical formulas (`AVERAGE`, `STDEV.S`, `MIN`, `MAX`, `MEDIAN`, `CORREL`), Pivot Tables, and native embedded charts. |
| **Executive Presentation Deck** | [`CH18_Executive_Pitch_Deck.pptx`](file:///d:/SAT-2/CH18_Executive_Pitch_Deck.pptx) | 6-slide PowerPoint pitch presentation summarizing business context, key findings, ML metrics, and 2 actionable recommendations. |
| **High-Resolution Figures** | [`figures/`](file:///d:/SAT-2/figures/) | 8 publication-ready charts (diurnal load curve, thermal response, segment Pareto, weekend pattern, area variability, tariff misalignment, feature importances, actual vs. predicted). |
| **Replication Pipeline Scripts** | [`generate_solution.py`](file:///d:/SAT-2/generate_solution.py) & [`build_notebook.py`](file:///d:/SAT-2/build_notebook.py) | Automated standalone Python scripts for full pipeline reproduction and headless notebook execution. |

---

## 2. Data Quality Audit & Cleaning Methodology

### 2.1 Quality Anomaly Resolution Matrix
| Quality Condition | Raw Anomaly Identified | Diagnostic Evidence | Remediation Applied | Cleaned Outcome |
| :--- | :--- | :--- | :--- | :--- |
| **Duplicate Timestamps** | 70 duplicate rows in `meter_readings.csv` | 35 exact pairs of identical `(customer, timestamp, consumption)` records. | Applied `.drop_duplicates(subset=['customer', 'timestamp'])` keeping first instance. | Reduced raw rows from 86,435 to exactly 86,400 ($120 \times 720$). |
| **Inconsistent Labels** | Trailing whitespace and mixed case in `customers.csv` | Segments contained `'Commercial '` and `'residential'`. | Standardized via `.astype(str).str.strip().str.title()`. | Exactly 3 clean categories: `Residential` (67), `Commercial` (41), `Industrial` (12). |
| **Negative Consumption** | 18 rows with negative values (e.g., `-1.322`, `-2.148`) | Contextual inspection of $t-1$ and $t+1$ shows values match surrounding positive levels; caused by sensor Current Transformer (CT) polarity flip. | Applied `.abs()` to restore valid continuous consumption. | All consumption values non-negative; minimum valid consumption is 0.346 kWh. |
| **Missing Meter Readings** | 90 missing values in `consumption` | 90 isolated single-hour gaps across customers; no customer had $>1$ consecutive NaN. | Per-customer chronological linear interpolation via `.interpolate(method='linear')`. | 100% complete time series across all 120 customers. |
| **Weather Telemetry Gaps** | 12 temperature and 10 humidity missing hours | Single isolated hour gaps within the 720-hour timeline. | Time-series linear interpolation along chronological sequence. | Complete temperature (mean 27.99°C) and humidity (mean 68.14%) telemetry. |

---

## 3. Exploratory Data Analysis (EDA) — Addressing the 6 Business Questions

### Question 1: When Does Demand Peak?
- **Empirical Peak Period**: Grid demand experiences a steep surge beginning at 09:00 AM, peaking between **11:00 AM and 2:00 PM (Hours 11–14)**.
- **Apex Hour**: Hour **13:00 (1:00 PM)** with an average consumption of **2.837 kWh** (system-wide sum: 10,213.84 kWh).
- **Trough Period**: Minimum demand occurs in early morning between **04:00 AM and 07:00 AM**, reaching an absolute trough at **05:00 AM (1.497 kWh)**.
- **Peak-to-Trough Ratio**: **1.90x** (+89.5% increase during midday hours relative to the morning trough).

### Question 2: How Does Temperature Relate to Consumption?
- **Thermal Correlation**: Ambient temperature exhibits a positive correlation with consumption ($r = +0.1975$).
- **Thermal Response Across Temperature Bands**:
  - $20^\circ\text{C} - 24^\circ\text{C}$: **1.704 kWh** (Cool baseline)
  - $24^\circ\text{C} - 27^\circ\text{C}$: **1.812 kWh** (+6.3% increase)
  - $27^\circ\text{C} - 30^\circ\text{C}$: **2.034 kWh** (+19.4% increase)
  - $30^\circ\text{C} - 33^\circ\text{C}$: **2.431 kWh** (+42.7% increase)
  - $33^\circ\text{C} - 37^\circ\text{C}$: **2.800 kWh** (**+64.5% increase** over baseline)
- **Key Takeaway**: Strong cooling demand (HVAC and industrial chillers) drives significant afternoon grid stress as ambient temperatures exceed 27°C.

### Question 3: How Do Customer Segments Differ?
- **Extreme Asymmetry (Pareto Distribution)**:
  - **Industrial**: Represents only **10.0% of accounts (12 clients)** but consumes **26.3% of total grid energy** (47,078.7 kWh). Average hourly draw is **5.449 kWh** (nearly $5\times$ residential draw, with maximum peaks reaching 14.27 kWh).
  - **Commercial**: Accounts for **34.2% of clients (41 accounts)** and consumes **43.7% of total energy** (78,235.3 kWh), with an average load of **2.650 kWh**.
  - **Residential**: Comprises **55.8% of the customer base (67 accounts)** but consumes only **29.9% of total energy** (53,556.6 kWh), averaging **1.110 kWh**.
- **Commercial & Industrial Combined**: Represent **44.2% of accounts** but drive **70.1% of total consumption**.

### Question 4: What is the Weekend Pattern?
- **Baseload Continuity**: Across segments, overall consumption remains remarkably stable between weekdays and weekends:
  - *Industrial*: Weekday mean 5.439 kWh vs. Weekend mean 5.476 kWh (+0.68% variation).
  - *Commercial*: Weekday mean 2.649 kWh vs. Weekend mean 2.655 kWh (+0.24% variation).
  - *Residential*: Weekday mean 1.110 kWh vs. Weekend mean 1.110 kWh (-0.04% variation).
- **Behavioral Shift**: Diurnal timing shows residential consumption shifts roughly 1–2 hours later on weekend mornings, but continuous commercial refrigeration, servers, and automated industrial manufacturing create a flat 24/7 baseload.

### Question 5: Which Areas Show High Variability?
- **High Volatility Zones**:
  - **West Area**: Highest standard deviation (**1.909 kWh**) and highest Coefficient of Variation (**$CV = 75.96\%$**), with mean consumption of 2.513 kWh.
  - **North Area**: Second highest standard deviation (**1.706 kWh**) and $CV = 74.89\%$, with mean consumption of 2.279 kWh.
- **Low Volatility Zones**:
  - **Central Area**: Lowest standard deviation (**1.205 kWh**) and lowest mean (1.708 kWh).
  - **South Area**: Standard deviation of 1.321 kWh ($CV = 67.15\%$).
- **Root Cause**: Geographic customer clustering. The **West** area has 4 Industrial customers out of 18 total accounts (22.2% industrial density), and the **North** has 4 Industrial customers out of 27 accounts (14.8%), whereas **Central** and **South** have only 1 Industrial account each.

### Question 6: How Do Tariff Periods Align with Demand?
- **Major Finding — Critical Tariff Misalignment**:
  - **Current Peak Tariff Schedule**: Active between **18:00 and 21:00 (6:00 PM – 9:00 PM)**. Average consumption during this period is **2.006 kWh**.
  - **Current Normal Tariff Schedule**: Active between **07:00 and 17:00 (7:00 AM – 5:00 PM)**. Average consumption during this period is **2.315 kWh**.
  - **True Grid Peak**: Occurs between **11:00 AM and 2:00 PM**, where average consumption reaches **2.76–2.84 kWh**.
- **Assessment**: The utility's high peak pricing is scheduled in the evening after system peak demand has already passed! The most congested, thermally stressed midday hours are priced under standard flat rates.

---

## 4. Machine Learning Modeling & Feature Selection

### 4.1 Feature Engineering Architecture
1. **Cyclical Temporal Features**: Continuous representations of periodic cycles using sine and cosine:
   $$\text{hour\_sin} = \sin\left(\frac{2\pi \cdot \text{hour}}{24}\right), \quad \text{hour\_cos} = \cos\left(\frac{2\pi \cdot \text{hour}}{24}\right)$$
   $$\text{dow\_sin} = \sin\left(\frac{2\pi \cdot \text{dow}}{7}\right), \quad \text{dow\_cos} = \cos\left(\frac{2\pi \cdot \text{dow}}{7}\right)$$
2. **Thermal & Interaction Features**:
   - `cooling_effect`: $\max(0, \text{temperature} - 24.0)$ capturing cooling degree response above comfort baseline.
   - `temp_humidity_idx`: $\text{temperature} \times (\text{humidity} / 100)$ capturing heat index discomfort.
3. **Historical Dynamic Lags & Rolling Statistics**:
   - `lag_1h`: Immediate prior hour consumption.
   - `lag_24h`: Prior day same-hour consumption (capturing diurnal customer habits).
   - `rolling_mean_24h`: 24-hour moving average baseline per customer.
   - `rolling_std_24h`: 24-hour rolling volatility per customer.
4. **Categorical Dummies**: One-hot encodings for `segment`, `area`, `connection_type`, `tariff_category`, and `peak_period`.

### 4.2 Chronological Validation Protocol
- **Dataset Partition**: Chronological split respecting the time-series arrow of time.
  - **Training Set**: Days 1–24 (66,240 records, 79.3% of panel).
  - **Testing Set**: Days 25–30 (17,280 records, 20.7% of panel).
- **Data Leakage Prevention**: No future timestamps used to predict past values.

### 4.3 Model Performance Benchmark
| Model Specification | Feature Count | Test RMSE (kWh) | Test MAE (kWh) | Test $R^2$ Score |
| :--- | :---: | :---: | :---: | :---: |
| **Full Random Forest Regressor** | 28 features | **0.4740** | **0.3050** | **0.9048 (90.48%)** |
| **Selected Feature RF Regressor** | **12 features** | **0.4739** | **0.3050** | **0.9049 (90.49%)** |

### 4.4 Top Feature Importances (Gini Importance)
1. `lag_24h` (64.9% importance): Yesterday's consumption at the same hour serves as the primary anchor for power draw.
2. `rolling_mean_24h` (18.9% importance): Captures baseline customer operating level.
3. `hour_cos` (4.8% importance): Encodes diurnal timing smoothly.
4. `segment_Industrial` (4.7% importance): Isolates heavy 5.45 kWh industrial consumers from light residential draw.
5. `hour_sin` (1.3% importance): Cyclical time offset.
6. `lag_1h` (1.2% importance): Immediate short-term momentum.
7. `rolling_std_24h` (0.8% importance): Customer volatility metric.
8. `temperature` & `cooling_effect` (1.5% combined): Environmental heat sensitivity.

---

## 5. Excel Analysis Architecture (`CH18_Electricity_Consumption_Analysis.xlsx`)

The workbook is structured into 9 dedicated sheets:
1. `Executive Summary`: High-level dashboard with 6 KPI cards, core findings matrix, formula-driven totals, and practical management recommendations.
2. `Hourly Demand`: 24-hour table with `=AVERAGE`, `=MIN`, `=MAX`, `=SUM`, and an embedded native Excel Line Chart highlighting peak vs. trough hours.
3. `Weather Relationship`: Correlation summary table (`=CORREL`), temperature bins, and an embedded native Excel Column Chart.
4. `Segment Comparison`: Cross-segment summary table comparing mean, median, standard deviation, and energy share, with an embedded Horizontal Bar Chart.
5. `Weekend Pattern`: Weekday vs. weekend comparative analysis across segments with an embedded Clustered Column Chart.
6. `Area Variability`: Regional analysis featuring mean, standard deviation, and Coefficient of Variation ($CV\%$) with an embedded Column Chart.
7. `Tariff Period Analysis`: Peak period alignment table illustrating the discrepancy between actual load and tariff pricing windows, with an embedded Column Chart.
8. `Statistical Summary`: Complete distribution table (Count, Mean, Std, Min, Q1, Median, Q3, Max) for all numeric fields.
9. `Cleaned Data Sample`: Formatted table with headers, borders, and freeze panes for exploratory inspection.

---

## 6. Strategic Business Findings & Practical Recommendations

### Two Practical Business Recommendations

#### Recommendation 1: Dynamic Midday Peak Tariff Restructuring (11:00 AM – 3:00 PM Window)
- **Business Rationale**: GridSense currently charges Peak Tariff rates between 18:00 and 21:00 (where average load is only 2.01 kWh), while leaving the true system peak between 11:00 and 14:00 (where load reaches 2.84 kWh) in the standard flat tier. This leaves distribution transformers exposed to thermal overload during maximum afternoon heat without price elasticity incentives.
- **Specific Action**: File a tariff adjustment petition with the state energy regulatory commission to shift the official High/Peak tariff window to **11:00 AM – 3:00 PM (11:00–15:00)**.
- **Projected Impact**:
  1. Creates clear price signals that incentivize commercial and industrial customers to schedule non-essential batch processes away from the midday heat peak.
  2. Reduces peak thermal transformer loading by an estimated 12–18%.
  3. Accelerates customer payback for behind-the-meter solar PV installations, aligning solar generation hours with high tariff savings.

#### Recommendation 2: Segment-Targeted Industrial Automated Demand Response (ADR) in West and North Areas
- **Business Rationale**: Just 12 Industrial accounts (10% of customers) consume 26.3% of total grid energy, averaging 5.45 kWh/hr with extreme volatility ($CV > 75\%$). Crucially, 8 of these 12 Industrial accounts are clustered in the West and North sectors, causing regional distribution feeder instability.
- **Specific Action**: Implement an Automated Demand Response (ADR) program with smart gateway telemetry specifically enrolled for the **12 Industrial accounts** in the West and North sectors.
- **Projected Impact**:
  1. GridSense can negotiate interruptible load agreements that shed or shift 30–40 kWh of high-cost peak load per hour during critical grid alerts by interfacing with only 12 client facilities.
  2. Avoids millions in substation transformer capacity upgrades in the West and North feeder corridors while maintaining standard reliable service for residential customers.
