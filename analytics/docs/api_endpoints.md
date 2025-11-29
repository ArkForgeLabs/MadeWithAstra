# API Endpoints Documentation

This document provides detailed information about all API endpoints available in the Analytics Server.

## Event Tracking Endpoints

### POST /apiv1/events

Ingest a single event.

#### Request Body

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

#### Response

On success:

```json
{
  "success": true,
  "message": "Event recorded"
}
```

On error:

```json
{
  "error": "string"
}
```

### POST /apiv1/events/batch

Ingest multiple events in batch.

#### Request Body

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

#### Response

On success:

```json
{
  "success": true,
  "results": [{ "success": true }, { "success": true }]
}
```

On error:

```json
{
  "error": "string"
}
```

## A/B Testing Endpoints

### POST /apiv1/ab-tests

Create a new A/B test.

#### Request Body

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

#### Response

On success:

```json
{
  "success": true,
  "test_id": 1
}
```

On error:

```json
{
  "error": "string"
}
```

### GET /apiv1/ab-tests

List all A/B tests.

#### Response

```json
{
  "tests": [
    {
      "id": 1,
      "test_name": "string",
      "description": "string",
      "created_at": "2023-01-01T00:00:00Z",
      "variants": [
        {
          "id": 1,
          "variant_name": "string",
          "description": "string",
          "weight": 1.0
        }
      ]
    }
  ]
}
```

### GET /apiv1/ab-tests/{test_id}

Get details of a specific A/B test.

#### Response

```json
{
  "id": 1,
  "test_name": "string",
  "description": "string",
  "created_at": "2023-01-01T00:00:00Z",
  "variants": [
    {
      "id": 1,
      "variant_name": "string",
      "description": "string",
      "weight": 1.0
    }
  ]
}
```

### POST /apiv1/ab-tests/{test_id}/assign

Assign a user to a variant.

#### Request Body

```json
{
  "user_id": "string"
}
```

#### Response

On success:

```json
{
  "user_id": "string",
  "test_id": 1,
  "variant": "string"
}
```

On error:

```json
{
  "error": "string"
}
```

### POST /apiv1/ab-tests/{test_id}/track

Track a metric for an A/B test.

#### Request Body

```json
{
  "user_id": "string",
  "metric_name": "string",
  "value": 1.0
}
```

#### Response

On success:

```json
{
  "success": true,
  "message": "Metric tracked"
}
```

On error:

```json
{
  "error": "string"
}
```

### GET /apiv1/ab-tests/{test_id}/results

Get results for an A/B test.

#### Response

```json
{
  "test_name": "string",
  "description": "string",
  "results": [
    {
      "variant_name": "string",
      "metric_count": 100,
      "average_value": 0.15,
      "min_value": 0.05,
      "max_value": 0.25,
      "metrics": [
        {
          "metric_name": "string",
          "value": 1.0,
          "created_at": "2023-01-01T00:00:00Z"
        }
      ]
    }
  ]
}
```

## Error Handling

All endpoints return appropriate HTTP status codes:

- `200 OK` - Successful GET requests
- `201 Created` - Successful POST requests
- `400 Bad Request` - Invalid request data
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server-side errors

## Authentication and Security

The analytics server does not include built-in authentication. For production use, consider implementing authentication middleware or using a reverse proxy with authentication.

## Rate Limiting

The server does not implement rate limiting by default. For high-volume applications, consider adding rate limiting to prevent abuse.

## Example Usage

### Tracking Events with JavaScript

```javascript
// Single event
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

// Batch events
fetch("http://localhost:20000/apiv1/events/batch", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify([
    {
      event_type: "page_view",
      user_id: "user123",
      session_id: "session456",
      properties: {
        page: "homepage",
      },
    },
  ]),
});
```

### A/B Testing with JavaScript

```javascript
// Create A/B test
fetch("http://localhost:20000/apiv1/ab-tests", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    test_name: "homepage_button_color",
    description: "Testing different button colors",
    variants: [
      {
        variant_name: "red",
        weight: 0.5,
      },
      {
        variant_name: "blue",
        weight: 0.5,
      },
    ],
  }),
});

// Assign user to variant
fetch("http://localhost:20000/apiv1/ab-tests/1/assign", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    user_id: "user123",
  }),
});

// Track metric
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
