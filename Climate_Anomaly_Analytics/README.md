# Multi-Stage Climate Anomaly Analytics: Sequential CTEs & Windows (MySQL)

> **Portfolio Project Context:** This project highlights my ability to construct clean, modular data pipelines using sequential Common Table Expressions (CTEs) and advanced window functions to extract chronological patterns and localized anomalies from raw environmental datasets. 

## 📌 Business Scenario

AgTech insurance underwriters and sustainability funds require high-fidelity automated pipelines to detect sudden, severe micro-climate anomalies. A critical risk metric for premium adjustments is the **Precipitation Acceleration Event (PAE)**. A PAE is defined as a specific day where a weather station records rainfall that drastically exceeds the localized historical regional monthly baseline, while compounding an aggressive day-over-day accelerating trend. 

The objective of this project is to build an optimized, multi-stage aggregation data pipeline in MySQL. The script ingests raw daily weather logs, computes rolling spatial-temporal benchmarks, and isolates high-risk accelerating anomalies for corporate asset tracking. 

## 📊 SQL Approach & Logic

To successfully compute valid annual risk metrics across an unstructured legacy dataset, the query executes the following programmatic phases: 

* **Modular Architecture (Sequential CTEs):** An optimized staging Common Table Expression isolates the analytical window execution layer, keeping the final reporting filter clean, maintainable, and human-readable.
* **Single-Scan Window Execution (AVG() OVER):** Calculates the average monthly precipitation for each distinct geographic region (region_code), using DATE_FORMAT(log_date, '%Y-%m') to prevent multi-year data overlap. This baseline is appended to every single daily log without collapsing the granular row structure.
* **Time-Series Lag Navigation (LAG() OVER):** Traverses chronological daily records partitioned strictly per individual weather station (station_id) to isolate the prior day's precipitation, demonstrating time-series data wrangling capabilities.
* **Multi-Condition Risk Filtering (WHERE):** The final outer query isolates specific climate records that fulfill both corporate risk metrics: current day's rainfall is **more than twice (2x)** the calculated regional monthly average, and simultaneously represents an accelerating trend of **at least 50% higher** (>= 1.5 * prev_day_precipitation) compared to the previous day.

## 💻 Optimized MySQL Query

sql

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

Use code with caution.

## 🎯 Key Takeaways & Skills Demonstrated

* Advanced execution of sequential transformation steps to separate raw database ingestion from analytical evaluation layers.
* Expertise in advanced window layout design, replacing expensive table self-joins with a high-performance single table scan architecture.
* Full mastery of cross-row analytical traversal functions (LAG) for micro-climate time-series modeling.
