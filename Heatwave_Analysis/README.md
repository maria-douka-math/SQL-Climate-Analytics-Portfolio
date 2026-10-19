# Climate Risk Analytics: High-Frequency Heatwave Anomaly Detection

## 📌 Business Scenario
Agricultural insurers, green energy grids, and supply chain managers rely heavily on historical weather patterns to mitigate financial risk. For this project, the objective is to isolate specific meteorological stations in Greece that recorded sustained climate stress during **2024**. 

Management requires a report identifying regions that experienced a severe heatwave anomaly, defined as tracking more than **500 total hourly observations above 35°C**. This insight directly guides infrastructure reinforcement and crop insurance premium adjustments.

## 📊 SQL Approach & Logic
To parse through large-scale, raw meteorological time-series databases efficiently, the following relational optimization techniques were deployed:
- **Time-Series Filtering:** A `WHERE` clause extracts observations strictly from the calendar year `2024` and filters the floating-point metrics to capture instances where `recorded_temperature > 35.0`°C.
- **Data Aggregation:** The dataset is aggregated using a multi-column `GROUP BY` on `station_id` and the extracted `observation_year` to evaluate localized trends.
- **Threshold Metric Monitoring:** Instead of row-level metrics, the `HAVING` clause uses `COUNT(reading_id)` to isolate areas suffering from chronic heat stress (> 500 hours) rather than brief spikes.
- **Sorting:** Orders results in descending sequence (`DESC`) to rank stations by their vulnerability profile.

## 💻 Optimized SQL Query
```sql
SELECT 
    ms.station_id, 
    YEAR(ms.reading_timestamp) AS observation_year, 
    COUNT(ms.reading_id) AS total_extreme_hours
FROM 
    meteorological_station_records ms
WHERE 
    YEAR(ms.reading_timestamp) = 2024 
    AND ms.recorded_temperature > 35.0
GROUP 
    BY ms.station_id, 
    YEAR(ms.reading_timestamp)
HAVING 
    COUNT(ms.reading_id) > 500
ORDER 
    BY total_extreme_hours DESC;
```

## 🎯 Key Takeaways & Skills Demonstrated
- Aggregating dense time-series metrics without performance bottlenecks.
- Dissecting logical operations between conditional string/date filters (`WHERE`) and aggregated thresholds (`HAVING`).
- Bridging academic environmental telemetry with corporate utility and operational risk workflows.
