# Multi-Stage Climate Anomaly Analytics (Sequential CTEs & Windows)

> **Domain:** AgTech Insurance & Climate Risk Analytics  
> **SQL Concepts:** Sequential Data Pipelines, Structural Window Functions (`AVG() OVER`), Time-Series Navigation (`LAG() OVER`), Multi-Condition Risk Filtering  

## 1. Business Scenario
AgTech insurance underwriters and sustainability funds require high-fidelity automated pipelines to detect sudden, severe micro-climate anomalies. A critical risk metric for premium adjustments is the **Precipitation Acceleration Event (PAE)**. A PAE is defined as a specific day where a weather station records rainfall that drastically exceeds the localized historical regional monthly baseline, while compounding an aggressive day-over-day accelerating trend.

The objective of this project is to build an optimized, multi-stage aggregation data pipeline in MySQL. The script ingests raw daily weather logs, computes rolling spatial-temporal benchmarks, and isolates high-risk accelerating anomalies for corporate asset tracking.

## 2. Business Logic & Technical Requirements
To minimize memory overhead and eliminate redundant table scans, the pipeline executes all transformations inside a single unified window execution stage before applying corporate thresholds:
1. **Regional Monthly Baseline:** Calculates the average monthly precipitation for each distinct geographic region (`region_code`), using `DATE_FORMAT(log_date, '%Y-%m')` to prevent multi-year data overlap. This baseline is appended to every single daily log without collapsing the granular row structure.
2. **Time-Series Lag Navigation:** Traverses chronological daily records partitioned strictly per individual weather station (`station_id`) to isolate the prior day's precipitation.
3. **Multi-Condition Risk Filtering:** Isolates specific climate records that fulfill both corporate risk metrics:
   * Current day's rainfall is **more than twice (2x)** the calculated regional monthly average.
   * Current day's rainfall represents an accelerating trend of **at least 50% higher** (`>= 1.5 * prev_day_precipitation`) compared to the previous day.

## 3. SQL Production Code
```sql
WITH time_and_regional_metrics AS (
    SELECT 
        station_id,
        region_code,
        log_date,
        daily_precipitation,
        -- 1. Compute rolling regional monthly baseline safely partitioned by specific year-month
        AVG(daily_precipitation) OVER(
            PARTITION BY region_code, DATE_FORMAT(log_date, '%Y-%m')
        ) AS regional_monthly_avg,
        -- 2. Fetch chronological previous day's value for the isolated weather station
        LAG(daily_precipitation) OVER(
            PARTITION BY station_id 
            ORDER BY log_date
        ) AS prev_day_precipitation
    FROM weather_sensor_logs
)
-- 3. Extract and filter high-risk Acceleration Events based on corporate criteria
SELECT 
    station_id,
    region_code,
    log_date,
    daily_precipitation,
    regional_monthly_avg,
    prev_day_precipitation
FROM time_and_regional_metrics
WHERE daily_precipitation > 2 * regional_monthly_avg
  AND daily_precipitation >= 1.5 * prev_day_precipitation
ORDER BY log_date ASC, station_id ASC;
```

## 4. Strategic Portfolio Takeaways
* **Single-Scan Architecture:** Optimized execution speed by embedding multiple analytical window functions within a single staging CTE. This approach prevents expensive self-joins, cutting computational database cost in half.
* **Chronological Granularity:** Successfully demonstrated execution of advanced analytical queries (`LAG`) on time-series meteorological logs while preserving structural multi-station partitioning logic.

