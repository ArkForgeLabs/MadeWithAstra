# Event Tracking Documentation

The event tracking system allows you to record user interactions and activities within your application. This functionality is essential for understanding user behavior, measuring engagement, and improving the overall user experience.

## Overview

The event tracking system provides two main endpoints for recording events:

- Single event ingestion via POST `/apiv1/events`
- Batch event ingestion via POST `/apiv1/events/batch`

## Event Structure

Each event consists of the following fields:

| Field        | Type   | Required | Description                                                          |
| ------------ | ------ | -------- | -------------------------------------------------------------------- |
| `event_type` | string | Yes      | The type of event being recorded (e.g., "button_click", "page_view") |
| `user_id`    | string | No       | Unique identifier for the user performing the action                 |
| `session_id` | string | No       | Identifier for the user's current session                            |
| `properties` | object | No       | Additional metadata about the event as key-value pairs               |

## Single Event Ingestion

### Endpoint

`POST /apiv1/events`

### Request Format

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

### Example Request

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

### Response

On successful creation, returns:

```json
{
  "success": true,
  "message": "Event recorded"
}
```

## Batch Event Ingestion

### Endpoint

`POST /apiv1/events/batch`

### Request Format

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

### Example Request

```javascript
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
        referrer: "https://example.com",
      },
    },
    {
      event_type: "button_click",
      user_id: "user123",
      session_id: "session456",
      properties: {
        button_name: "submit",
        page: "homepage",
      },
    },
  ]),
});
```

### Response

On successful creation, returns:

```json
{
  "success": true,
  "results": [{ "success": true }, { "success": true }]
}
```

## Best Practices

1. **Event Naming**: Use descriptive and consistent event names (e.g., "button_click", "form_submit", "page_view")
2. **User Identification**: Include `user_id` when available for user-level analytics
3. **Session Tracking**: Use `session_id` to group related events together
4. **Properties Usage**: Include relevant metadata in the properties field to provide context
5. **Batch Processing**: Use batch ingestion for multiple events to reduce network overhead

## Data Storage

Events are stored in the SQLite database with the following schema:

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

The `properties` field stores the event metadata as a JSON string for flexibility in storing different types of data.
