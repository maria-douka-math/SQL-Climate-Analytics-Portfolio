# Climate Risk Analytics: Thessaloniki Flash Flood Risk Profiling (MySQL)

### 📊 Portfolio Project Context
This project highlights my ability to construct clean, modular data pipelines using advanced Common Table Expressions (CTEs) and conditional aggregation models to isolate severe environmental risks from corrupted telemetry streams.

### 📌 Business Scenario
Urban planning departments and municipal civil protection authorities monitor annual precipitation thresholds to identify areas highly vulnerable to flash floods and cloudburst events. 

The objective of this project is to analyze legacy historical telemetry logs from 2026 across Thessaloniki's weather stations to isolate locations where the valid annual rainfall exceeded a critical threshold of 600mm. Because legacy sensors frequently experience electrical faults that log impossible negative values, the query must dynamically purge sensor noise to deliver an accurate, high-risk climate volatility report.

### 📊 SQL Approach & Logic
To successfully compute valid annual risk metrics across an unstructured legacy dataset, the query executes the following programmatic phases:

* **Modular Architecture (CTE):** A Common Table Expression (WITH ... AS) isolates the dense aggregation and data-cleaning footprint, keeping the final reporting layer clean and highly maintainable.
* **Relational Integrity (JOIN):** An INNER JOIN connects the localized station directory (thessaloniki_stations) with the dense tracking logs (precipitation_telemetry) using the shared station_id key.
* **Data Type Normalization (CAST):** Row-level pre-filtering converts unstructured string date fields (VARCHAR) into valid chronological formats using CAST(column AS DATE) to safely restrict processing to the calendar year 2026 via the YEAR() function.
* **Conditional Aggregation (CASE WHEN):** Programmatic cleaning is embedded directly within the SUM() aggregate function, capturing only realistic observations (>= 0) and replacing corrupted sensor noise with NULL to maintain strict statistical integrity.
* **Alias-Driven Extraction (WHERE):** The final outer query bypasses heavy post-aggregation HAVING clauses by filtering directly against the CTE's pre-computed metric alias (yearly_rainfall_mm > 600).

### 💻 Optimized MySQL Query

```sql
WITH yearly_rainfall_2026 AS (
    SELECT 
        stat.station_name,
        SUM(CASE 
            WHEN prec.daily_rainfall_mm >= 0 THEN prec.daily_rainfall_mm 
            ELSE NULL 
        END) AS yearly_rainfall_mm
    FROM thessaloniki_stations stat
    JOIN precipitation_telemetry prec ON stat.station_id = prec.station_id
    WHERE YEAR(CAST(prec.log_date_str AS DATE)) = 2026
    GROUP BY stat.station_name
)
SELECT 
    station_name, 
    yearly_rainfall_mm
FROM yearly_rainfall_2026
WHERE yearly_rainfall_mm > 600;
```

### 🎯 Key Takeaways & Skills Demonstrated
* **Advanced execution of Common Table Expressions (CTEs)** to separate data transformation logic from final business presentation layers.
* **Expertise in data cleaning and data quality engineering** through conditional aggregate handling (CASE WHEN inside SUM).
* **Successful execution of data type casting** on legacy string formats to guarantee temporal partition compliance.
