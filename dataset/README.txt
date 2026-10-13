CH18 — Electricity Consumption Insight (Practice Dataset Pack)

IMPORTANT:
These CSVs are SYNTHETIC / SIMULATED practice data generated from the CH18.pdf schema.
They are NOT the original datasets from the assessment provider. The PDF describes the
files and their fields but does not include the actual CSV contents.

Files:
- meter_readings.csv: timestamp, customer, consumption
- customers.csv: customer, segment, area, connection_type
- weather.csv: timestamp, temperature, humidity, rain
- calendar.csv: timestamp, weekday, holiday, season
- tariff.csv: timestamp, tariff_category, peak_period

Relationships:
- meter_readings.customer -> customers.customer
- timestamp in meter_readings -> timestamp in weather, calendar, and tariff

Data period:
- 2026-09-01 through 2026-09-30, hourly timestamps
- 120 customers
- Expected raw meter rows before injected duplicates: 120 x 720 = 86,400
- Quality issues intentionally included in meter_readings: missing consumption, duplicate rows,
  and negative consumption values.
- Weather includes missing temperature/humidity values.
- customers includes inconsistent capitalization/spacing in some segment labels.

Recommended cleaning:
1. Parse timestamps as datetime.
2. Standardize segment labels (trim spaces and consistent capitalization).
3. Identify duplicates using customer + timestamp; investigate before dropping.
4. Investigate negative consumption; do not automatically assume all are valid.
5. Handle missing meter readings and weather gaps with a documented method.
6. Merge tables using customer and timestamp keys.
7. Split train/test chronologically if forecasting consumption.

Because the data is synthetic, any EDA findings or ML metrics obtained from it must be reported as
practice results, not as official results from the original challenge dataset.
