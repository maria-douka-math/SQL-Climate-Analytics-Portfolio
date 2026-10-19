# Climate Risk Analytics: AUTH Meteorological Time-Series Optimization
> **Personal Project Context:** This analysis is directly inspired by my academic work and rotational shifts at the Aristotle University of Thessaloniki (AUTH) Meteorological Station, where I monitored and maintained database integrity for large-scale climate records.

## 📌 Business Scenario
Agricultural insurance agencies and local government bodies in Central Macedonia require precise tracking of extreme weather phenomena to evaluate crop damage risks and structural stress. 

Using the historical climate data framework from the **AUTH database**, this project isolates specific monitoring systems that registered severe heatwave anomalies during the **2024** calendar year. The business objective is to flag areas experiencing more than **500 cumulative hours of temperatures above 35°C** for targeted financial risk assessment.

## 📊 SQL Approach & Logic
To process the dense, hourly time-series tracking data, the query utilizes optimized relational filtering and aggregation:
- **AUTH Dataset Filtering:** The `WHERE` clause extracts timestamps matching the year `2024` and filters the floating-point temperature metrics where `hourly_temp > 35.0`°C.
- **Station Grouping:** A multi-column `GROUP BY` aggregates the metrics by `station_id` and the computed `observation_year`.
- **Anomalous Threshold Isolation:** The `HAVING` clause filters the aggregated rows using `COUNT(record_id)` to ensure only stations with chronic, high-frequency heat exposure are reported to risk managers.

## 💻 Optimized SQL Query
```sql
SELECT 
    auth.station_id, 
    YEAR(auth.reading_timestamp) AS observation_year, 
    COUNT(auth.record_id) AS total_extreme_hours
FROM 
    auth_meteo_warehouse auth
WHERE 
    YEAR(auth.reading_timestamp) = 2024 
    AND auth.hourly_temp > 35.0
GROUP 
    BY auth.station_id, 
    YEAR(auth.reading_timestamp)
HAVING 
    COUNT(auth.record_id) > 500
ORDER 
    BY total_extreme_hours DESC;
```

## 🎯 Key Takeaways & Skills Demonstrated
- Transforming raw academic environmental telemetry into structural corporate risk metrics.
- Mastery of the SQL order of execution by separating raw data filters (`WHERE`) from aggregate metrics (`HAVING`).
- Handling real-world, large-scale time-series records (500K+ records) with optimized indexing logic.
