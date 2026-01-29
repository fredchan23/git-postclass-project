Project: Daily Steps Tracker Database
Design a database to track daily walking activity.
Requirements:
Commit 1: Core table

Create DBML file with daily_steps table
Fields: date (PK), step_count, distance_km, duration_minutes
Add NOT NULL constraints

Commit 2: Goals table

Add weekly_goals table (week_start_date PK, target_steps)
Link daily_steps to weekly_goals via week relationship

Commit 3: Normalize activity details

Create activity_types lookup table (walking, hiking, jogging)
Add foreign key from daily_steps to activity_types
Document why this improves the schema

Commit 4: Visualization

Generate ERD using dbdiagram.io
Save diagram to repo as PNG

Commit 5: Documentation

README with schema explanation
List 3 example queries (average steps per week, goal achievement, etc.)