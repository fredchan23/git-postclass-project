# Daily Steps Activity Tracking Schema

## Schema Overview

This DBML schema tracks daily step metrics with activity classification and weekly goal management.

### Tables

**daily_steps** - Core fact table storing daily activity metrics
- `date` (PK): Unique date for each activity record
- `step_count`: Total steps taken on that day
- `distance_km`: Total distance covered in kilometers
- `duration_minutes`: Total activity duration in minutes
- `week_start_date` (FK): Links to the corresponding weekly goal period
- `activity_type_id` (FK): References the type of activity performed

**weekly_goals** - Dimension table for weekly step targets
- `week_start_date` (PK): Start date of the goal week
- `target_steps`: Target step count for that week

**activity_types** - Lookup table for activity classification
- `id` (PK): Auto-incrementing activity identifier
- `activity_name` (UNIQUE): Activity type (e.g., walking, hiking, jogging)

### Relationships
- `daily_steps` → `weekly_goals` via `week_start_date` (many-to-one)
- `daily_steps` → `activity_types` via `activity_type_id` (many-to-one)

---

## Example Queries

### 1. Average Steps Per Week
Calculate the average daily steps for each week and compare to goals:

```sql
SELECT 
  wg.week_start_date,
  wg.target_steps,
  ROUND(AVG(ds.step_count), 0) as avg_daily_steps,
  COUNT(ds.date) as days_recorded,
  CASE 
    WHEN AVG(ds.step_count) >= wg.target_steps THEN 'Goal Met'
    ELSE 'Goal Missed'
  END as status
FROM weekly_goals wg
LEFT JOIN daily_steps ds ON wg.week_start_date = ds.week_start_date
GROUP BY wg.week_start_date, wg.target_steps
ORDER BY wg.week_start_date DESC;
```

### 2. Goal Achievement Rate
Determine what percentage of weeks the user met their step targets:

```sql
SELECT 
  COUNT(*) as total_weeks,
  SUM(CASE WHEN avg_daily_steps >= target_steps THEN 1 ELSE 0 END) as weeks_achieved,
  ROUND(
    100.0 * SUM(CASE WHEN avg_daily_steps >= target_steps THEN 1 ELSE 0 END) / COUNT(*), 
    2
  ) as achievement_rate_percent
FROM (
  SELECT 
    wg.week_start_date,
    wg.target_steps,
    AVG(ds.step_count) as avg_daily_steps
  FROM weekly_goals wg
  LEFT JOIN daily_steps ds ON wg.week_start_date = ds.week_start_date
  GROUP BY wg.week_start_date, wg.target_steps
) weekly_stats;
```

### 3. Activity Type Performance
Compare average distance and duration across different activity types:

```sql
SELECT 
  at.activity_name,
  COUNT(ds.date) as total_days,
  ROUND(AVG(ds.step_count), 0) as avg_steps,
  ROUND(AVG(ds.distance_km), 2) as avg_distance_km,
  ROUND(AVG(ds.duration_minutes), 0) as avg_duration_minutes,
  ROUND(AVG(ds.distance_km) / AVG(ds.duration_minutes), 3) as pace_km_per_min
FROM daily_steps ds
JOIN activity_types at ON ds.activity_type_id = at.id
GROUP BY at.activity_name
ORDER BY total_days DESC;
```

---

## Key Schema Benefits

✓ **Data Integrity** - Foreign keys prevent invalid activity types  
✓ **Efficiency** - Lookup table reduces data redundancy  
✓ **Flexibility** - Easy to add new activity types  
✓ **Scalability** - Structure supports tracking multiple users with minimal modification  
✓ **Analytics Ready** - Normalized structure enables powerful aggregations
