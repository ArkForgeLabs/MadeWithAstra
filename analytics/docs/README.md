# Analytics Server Documentation

The Analytics Server is a component of the ArkForge Servitor project that provides comprehensive analytics capabilities including event tracking and A/B testing functionality.

## Features

- **Event Tracking**: High-performance endpoint for recording user events with metadata and properties
- **A/B Testing**: Full-featured A/B testing framework with weighted assignments and metric tracking
- **SQLite Database**: Persistent storage using SQLite for simplicity and performance
- **RESTful API**: Well-defined endpoints for integration with frontend and backend services

## Getting Started

The analytics server is automatically initialized when the main server starts. It runs on port 20000 and is accessible from the same host as the main server.

## Architecture

The analytics server consists of two main components:

1. **Event Tracking System** - Records user interactions and activities
2. **A/B Testing Framework** - Manages experiments and variant assignments

## Database

The server uses SQLite for data persistence with the following schema:

### Events Table

Stores user events with metadata and properties.

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

### A/B Tests Table

Stores A/B test configurations.

```sql
CREATE TABLE ab_tests (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    test_name TEXT NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### A/B Test Variants Table

Stores variants within A/B tests.

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

### A/B Test Assignments Table

Tracks which users are assigned to which variants.

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

### A/B Test Metrics Table

Stores metrics collected during A/B tests.

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

## API Endpoints

### Event Ingestion

#### POST /apiv1/events

Ingest a single event.

**Request Body:**

```json
{
  "event_type": "string",
  "user_id": "string",
  "session_id": "string",
  "properties": {
    "key": "value"
  }
}
```

#### POST /apiv1/events/batch

Ingest multiple events in batch.

**Request Body:**

```json
[
  {
    "event_type": "string",
    "user_id": "string",
    "session_id": "string",
    "properties": {
      "key": "value"
    }
  }
]
```

### A/B Testing

#### POST /apiv1/ab-tests

Create a new A/B test.

**Request Body:**

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

#### GET /apiv1/ab-tests

List all A/B tests.

#### GET /apiv1/ab-tests/{test_id}

Get details of a specific A/B test.

#### POST /apiv1/ab-tests/{test_id}/assign

Assign a user to a variant.

**Request Body:**

```json
{
  "user_id": "string"
}
```

#### POST /apiv1/ab-tests/{test_id}/track

Track a metric for an A/B test.

**Request Body:**

```json
{
  "user_id": "string",
  "metric_name": "string",
  "value": 1.0
}
```

#### GET /apiv1/ab-tests/{test_id}/results

Get results for an A/B test.

## Usage

The analytics server is automatically initialized when the main server starts. All endpoints are available on the same port as the main server (port 20000).

## Integration

To integrate with the analytics server from your frontend application:

1. Send events to `/apiv1/events` endpoint
2. Create A/B tests using `/apiv1/ab-tests` endpoint
3. Assign users to variants using `/apiv1/ab-tests/{test_id}/assign`
4. Track metrics using `/apiv1/ab-tests/{test_id}/track`
5. Retrieve results using `/apiv1/ab-tests/{test_id}/results`

## Example Usage

### Tracking an Event

```javascript
fetch("http://localhost:20000/apiv1/events", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    event_type: "button_click",
    user_id: "user123",
    session_id: "session456",
    properties: {
      button_name: "submit",
      page: "homepage",
    },
  }),
});
```

### Creating an A/B Test

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
