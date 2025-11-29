# Database Schema Documentation

The Analytics Server uses SQLite for data persistence with a well-defined schema that supports both event tracking and A/B testing functionality.

## Overview

The database consists of five main tables:

1. **events** - Stores user events with metadata and properties
2. **ab_tests** - Configuration for A/B tests
3. **ab_variants** - Variants within A/B tests
4. **ab_assignments** - Tracks which users are assigned to which variants
5. **ab_metrics** - Stores metrics collected during A/B tests

## Events Table

Stores user events with metadata and properties.

### Schema

```sql
CREATE TABLE events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    event_type TEXT NOT NULL,
    user_id TEXT,
    session_id TEXT,
    properties TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Fields

| Field        | Type                | Description                                       |
| ------------ | ------------------- | ------------------------------------------------- |
| `id`         | INTEGER PRIMARY KEY | Unique identifier for the event                   |
| `event_type` | TEXT NOT NULL       | Type of event (e.g., "button_click", "page_view") |
| `user_id`    | TEXT                | Unique identifier for the user                    |
| `session_id` | TEXT                | Identifier for the user's session                 |
| `properties` | TEXT                | JSON string containing additional event metadata  |
| `created_at` | TIMESTAMPTZ         | Timestamp when the event was recorded             |

## A/B Tests Table

Stores A/B test configurations.

### Schema

```sql
CREATE TABLE ab_tests (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    test_name TEXT NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Fields

| Field         | Type                 | Description                               |
| ------------- | -------------------- | ----------------------------------------- |
| `id`          | INTEGER PRIMARY KEY  | Unique identifier for the test            |
| `test_name`   | TEXT NOT NULL UNIQUE | Name of the A/B test                      |
| `description` | TEXT                 | Description of what the test is measuring |
| `created_at`  | TIMESTAMPTZ          | Timestamp when the test was created       |

## A/B Test Variants Table

Stores variants within A/B tests.

### Schema

```sql
CREATE TABLE ab_variants (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    test_id INTEGER NOT NULL,
    variant_name TEXT NOT NULL,
    description TEXT,
    weight REAL NOT NULL DEFAULT 1.0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (test_id) REFERENCES ab_tests(id) ON DELETE CASCADE
);
```

### Fields

| Field          | Type                      | Description                                        |
| -------------- | ------------------------- | -------------------------------------------------- |
| `id`           | INTEGER PRIMARY KEY       | Unique identifier for the variant                  |
| `test_id`      | INTEGER NOT NULL          | Foreign key to the parent A/B test                 |
| `variant_name` | TEXT NOT NULL             | Name of the variant (e.g., "control", "variant_a") |
| `description`  | TEXT                      | Description of what this variant represents        |
| `weight`       | REAL NOT NULL DEFAULT 1.0 | Weight for weighted random assignment              |
| `created_at`   | TIMESTAMPTZ               | Timestamp when the variant was created             |

## A/B Test Assignments Table

Tracks which users are assigned to which variants.

### Schema

```sql
CREATE TABLE ab_assignments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id TEXT NOT NULL,
    test_id INTEGER NOT NULL,
    variant_id INTEGER NOT NULL,
    assigned_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (test_id) REFERENCES ab_tests(id) ON DELETE CASCADE,
    FOREIGN KEY (variant_id) REFERENCES ab_variants(id) ON DELETE CASCADE,
    UNIQUE(user_id, test_id)
);
```

### Fields

| Field                      | Type                | Description                                             |
| -------------------------- | ------------------- | ------------------------------------------------------- |
| `id`                       | INTEGER PRIMARY KEY | Unique identifier for the assignment                    |
| `user_id`                  | TEXT NOT NULL       | Unique identifier for the user                          |
| `test_id`                  | INTEGER NOT NULL    | Foreign key to the A/B test                             |
| `variant_id`               | INTEGER NOT NULL    | Foreign key to the variant                              |
| `assigned_at`              | TIMESTAMPTZ         | Timestamp when the user was assigned                    |
| `UNIQUE(user_id, test_id)` | Constraint          | Ensures a user is assigned to only one variant per test |

## A/B Test Metrics Table

Stores metrics collected during A/B tests.

### Schema

```sql
CREATE TABLE ab_metrics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    test_id INTEGER NOT NULL,
    variant_id INTEGER NOT NULL,
    metric_name TEXT NOT NULL,
    value REAL NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (test_id) REFERENCES ab_tests(id) ON DELETE CASCADE,
    FOREIGN KEY (variant_id) REFERENCES ab_variants(id) ON DELETE CASCADE
);
```

### Fields

| Field         | Type                | Description                            |
| ------------- | ------------------- | -------------------------------------- |
| `id`          | INTEGER PRIMARY KEY | Unique identifier for the metric       |
| `test_id`     | INTEGER NOT NULL    | Foreign key to the A/B test            |
| `variant_id`  | INTEGER NOT NULL    | Foreign key to the variant             |
| `metric_name` | TEXT NOT NULL       | Name of the metric being tracked       |
| `value`       | REAL NOT NULL       | Value of the metric                    |
| `created_at`  | TIMESTAMPTZ         | Timestamp when the metric was recorded |

## Relationships

The tables are connected through foreign key relationships:

- `ab_variants.test_id` references `ab_tests.id`
- `ab_assignments.test_id` references `ab_tests.id`
- `ab_assignments.variant_id` references `ab_variants.id`
- `ab_metrics.test_id` references `ab_tests.id`
- `ab_metrics.variant_id` references `ab_variants.id`

## Indexes and Performance

The schema is designed for efficient querying:

- Primary keys are automatically indexed for fast lookups
- Foreign key relationships ensure data integrity
- Timestamps are used for time-based queries
- JSON fields are stored as text for flexibility

## Data Types

The schema uses SQLite's built-in data types:

- `INTEGER` for IDs and counts
- `TEXT` for strings and JSON data
- `REAL` for numeric values
- `TIMESTAMPTZ` for timestamps (using SQLite's datetime functions)

## Example Queries

### Get all events for a specific user

```sql
SELECT * FROM events WHERE user_id = 'user123' ORDER BY created_at DESC;
```

### Get all variants for a specific test

```sql
SELECT * FROM ab_variants WHERE test_id = 1;
```

### Get assignment details for a user

```sql
SELECT a.*, v.variant_name
FROM ab_assignments a
JOIN ab_variants v ON a.variant_id = v.id
WHERE a.user_id = 'user123';
```

### Get metrics for a specific test

```sql
SELECT v.variant_name, m.metric_name, m.value, m.created_at
FROM ab_metrics m
JOIN ab_variants v ON m.variant_id = v.id
WHERE m.test_id = 1
ORDER BY m.created_at DESC;
```
