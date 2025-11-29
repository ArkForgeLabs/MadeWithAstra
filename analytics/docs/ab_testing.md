# A/B Testing Documentation

The A/B testing framework allows you to run experiments to compare different versions of features and measure their impact on user behavior. This system provides a complete solution for creating, managing, and analyzing A/B tests.

## Overview

The A/B testing system provides the following core functionality:

- Create and manage A/B tests with multiple variants
- Assign users to different variants using weighted randomization
- Track metrics for each variant
- Analyze results to determine which variant performs better

## A/B Test Structure

An A/B test consists of:

- **Test Configuration**: Name, description, and metadata
- **Variants**: Different versions of the feature being tested
- **Assignments**: Which users are assigned to which variants
- **Metrics**: Quantitative data collected during the test

## API Endpoints

### Creating an A/B Test

#### Endpoint

`POST /apiv1/ab-tests`

#### Request Format

```json
{
  "test_name": "string",
  "description": "string",
  "variants": [
    {
      "variant_name": "string",
      "description": "string",
      "weight": 1.0
    }
  ]
}
```

#### Example Request

```javascript
fetch("http://localhost:20000/apiv1/ab-tests", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    test_name: "homepage_button_color",
    description: "Testing different button colors on homepage",
    variants: [
      {
        variant_name: "red",
        description: "Red button",
        weight: 0.5,
      },
      {
        variant_name: "blue",
        description: "Blue button",
        weight: 0.5,
      },
    ],
  }),
});
```

#### Response

```json
{
  "success": true,
  "test_id": 1
}
```

### Listing All A/B Tests

#### Endpoint

`GET /apiv1/ab-tests`

#### Response

```json
{
  "tests": [
    {
      "id": 1,
      "test_name": "homepage_button_color",
      "description": "Testing different button colors on homepage",
      "created_at": "2023-01-01T00:00:00Z",
      "variants": [
        {
          "id": 1,
          "variant_name": "red",
          "description": "Red button",
          "weight": 0.5
        },
        {
          "id": 2,
          "variant_name": "blue",
          "description": "Blue button",
          "weight": 0.5
        }
      ]
    }
  ]
}
```

### Getting Test Details

#### Endpoint

`GET /apiv1/ab-tests/{test_id}`

#### Response

```json
{
  "id": 1,
  "test_name": "homepage_button_color",
  "description": "Testing different button colors on homepage",
  "created_at": "2023-01-01T00:00:00Z",
  "variants": [
    {
      "id": 1,
      "variant_name": "red",
      "description": "Red button",
      "weight": 0.5
    },
    {
      "id": 2,
      "variant_name": "blue",
      "description": "Blue button",
      "weight": 0.5
    }
  ]
}
```

### Assigning Users to Variants

#### Endpoint

`POST /apiv1/ab-tests/{test_id}/assign`

#### Request Format

```json
{
  "user_id": "string"
}
```

#### Example Request

```javascript
fetch("http://localhost:20000/apiv1/ab-tests/1/assign", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    user_id: "user123",
  }),
});
```

#### Response

```json
{
  "user_id": "user123",
  "test_id": 1,
  "variant": "red"
}
```

### Tracking Metrics

#### Endpoint

`POST /apiv1/ab-tests/{test_id}/track`

#### Request Format

```json
{
  "user_id": "string",
  "metric_name": "string",
  "value": 1.0
}
```

#### Example Request

```javascript
fetch("http://localhost:20000/apiv1/ab-tests/1/track", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    user_id: "user123",
    metric_name: "conversion_rate",
    value: 0.15,
  }),
});
```

#### Response

```json
{
  "success": true,
  "message": "Metric tracked"
}
```

### Getting Test Results

#### Endpoint

`GET /apiv1/ab-tests/{test_id}/results`

#### Response

```json
{
  "test_name": "homepage_button_color",
  "description": "Testing different button colors on homepage",
  "results": [
    {
      "variant_name": "red",
      "metric_count": 100,
      "average_value": 0.15,
      "min_value": 0.05,
      "max_value": 0.25,
      "metrics": [
        {
          "metric_name": "conversion_rate",
          "value": 0.15,
          "created_at": "2023-01-01T00:00:00Z"
        }
      ]
    },
    {
      "variant_name": "blue",
      "metric_count": 120,
      "average_value": 0.18,
      "min_value": 0.08,
      "max_value": 0.28,
      "metrics": [
        {
          "metric_name": "conversion_rate",
          "value": 0.18,
          "created_at": "2023-01-01T00:00:00Z"
        }
      ]
    }
  ]
}
```

## Weighted Assignment

Users are assigned to variants using weighted randomization. The weight determines the probability of a user being assigned to a specific variant:

- If you have two variants with weights 0.5 and 0.5, each variant has a 50% chance of being selected
- If you have two variants with weights 0.7 and 0.3, the first variant has a 70% chance of being selected

## Best Practices

1. **Clear Test Names**: Use descriptive names that clearly indicate what's being tested
2. **Appropriate Weights**: Set weights based on your confidence in each variant
3. **Sufficient Sample Size**: Ensure enough users are assigned to get statistically significant results
4. **Metric Selection**: Choose meaningful metrics that directly relate to your business goals
5. **Test Duration**: Run tests long enough to collect sufficient data but not so long as to miss important changes

## Data Storage

A/B tests are stored in the SQLite database with the following schema:

```sql
CREATE TABLE ab_tests (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    test_name TEXT NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE ab_variants (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    test_id INTEGER NOT NULL,
    variant_name TEXT NOT NULL,
    description TEXT,
    weight REAL NOT NULL DEFAULT 1.0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (test_id) REFERENCES ab_tests(id) ON DELETE CASCADE
);

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
