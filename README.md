
# Microservices:
- User Management Service (UMS): Handles user authentication & authorization.
- Task Management Service (TMS): Manages task-related operations.
- Portal API Gateway (PGW): Unified entry point for routing requests.
- Database: NoSQL (MongoDB) for flexibility and scalability.
- Event Streaming: Kafka (if needed for async task updates/notifications).
- Logging & Monitoring: Loki + Grafana for observability.

---

# Micro Services
- User Management Server (UMS)
  - Sign on
  - Sign in
  - Sign out
- Task Management Server (TMS)
  - Create Task
  - Update Task
  - Delete Task
  - Get Task
  - Search Task

---
# API Spec
## User Management Server (UMS)
### Sign on
Method: `POST`

Path: `/api/user`

Headers:
`Content-Type: application/json`
  
Request Body:
```
{
  "username": "johndoe",
  "password": "securepass123",
  "email": "johndoe@example.com"
}
```

Response Body:
```
{
  "id": "user_123",
  "username": "johndoe",
  "email": "johndoe@example.com",
  "token": "jwt-token"
}
```
Errors:
- `400 Bad Request` (Invalid input)
- `409 Conflict` (Username/email already exists)

### Sign in
Method: `POST`

Path: `/api/user/login`

Headers:
`Content-Type: application/json`
  
Request Body:
```
{
  "username": "johndoe",
  "password": "securepass123"
}
```

Response Body:
```
{
  "token": "jwt-token"
}
```
Errors:
- `401 Unauthorized` (Invalid credentials)

### Sign out
Method: `POST`

Path: `/api/user/logout`

Headers:
`Authorization: Bearer <jwt-token>`

Response:
- `204 No Content`

---

## Task Management Server (TMS)
### Create Task
Method: `POST`

Path: `/api/tasks`

Headers:
`Authorization: Bearer <jwt-token>`

Request Body:
```
{
  "title": "Buy groceries",
  "description": "Milk, Eggs, Bread",
  "due_date": "2025-04-10T12:00:00Z",
  "priority": "high"
}
```

Response Body:
```
{
  "id": "task_456",
  "title": "Buy groceries",
  "status": "pending",
  "created_at": "2025-03-20T10:00:00Z"
}
```
Errors:
- `400 Bad Request` (Invalid input)
- `401 Unauthorized` (Invalid token)

### Update Task
Method: `PUT`

Path: `/api/tasks/{task_id}`

Headers:
`Authorization: Bearer <jwt-token>`

Request Body:
```
{
  "title": "Buy groceries",
  "status": "completed"
}
```

Response Body:
```
{
  "id": "task_456",
  "title": "Buy groceries",
  "status": "completed"
}
```
Errors:
- `400 Bad Request` (Invalid input)
- `404 Not Found` (Task not found)
- `401 Unauthorized` (Invalid token)

### Delete Task
Method: `DELETE`

Path: `/api/tasks/{task_id}`

Headers:
`Authorization: Bearer <jwt-token>`

Response:
- `204 No Content`

Errors:
- `404 Not Found` (Task not found)
- `401 Unauthorized` (Invalid token)

### Get Task
Method: `GET`

Path: `/api/tasks/{task_id}`

Headers:
`Authorization: Bearer <jwt-token>`

Response Body:
```
{
  "id": "task_456",
  "title": "Buy groceries",
  "status": "pending"
}
```
Errors:
- `404 Not Found` (Task not found)
- `401 Unauthorized` (Invalid token)

### Search Task
Method: `GET`

Path: `/api/tasks?status=pending&priority=high`

Headers:
`Authorization: Bearer <jwt-token>`

Response Body:
```
[
  {
    "id": "task_456",
    "title": "Buy groceries",
    "status": "pending"
  }
]
```
Errors:
- `401 Unauthorized` (Invalid token)

---
# Database Design (MongoDB Schema)
User Collection
```
{
  "_id": UUID("550e8400-e29b-41d4-a716-446655440000"),
  "username": "berkelee",
  "password_hash": "$2a$12$...",
  "email": "berkelee@example.com",
  "status": "active",
  "created_at": ISODate("2025-03-20T10:00:00Z"),
  "updated_at": ISODate("2025-03-20T12:00:00Z"),
  "last_login_at": ISODate("2025-03-20T12:30:00Z")
}
```

Task Collection
```
{
  "_id": UUID("661e8400-e29b-41d4-a716-446655440111"),
  "user_id": UUID("550e8400-e29b-41d4-a716-446655440000"),
  "status": "pending",
  "task_info": {
    "name": "Buy groceries",
    "content": "Milk, Eggs, Bread"
  },
  "created_at": ISODate("2025-03-20T10:05:00Z"),
  "updated_at": ISODate("2025-03-20T11:00:00Z")
}
```
## Indexing Strategy for Performance

| Field(s)                      | Index Type         | Purpose                                      |
|--------------------------------|-------------------|----------------------------------------------|
| `username` (unique)           | Single Index      | Fast username lookup                        |
| `email` (unique)              | Single Index      | Fast email lookup                           |
| `status`                      | Single Index      | Efficient filtering of active users         |
| `user_id` in `tasks`          | Single Index      | Fast retrieval of tasks for a user          |
| `status, created_at` in `tasks` | Compound Index  | Fast filtering of tasks by status & date    |
| `updated_at` in `tasks`        | TTL Index (Optional) | Auto-delete old tasks after a period  |



