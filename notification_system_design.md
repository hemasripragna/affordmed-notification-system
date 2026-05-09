# Notification System Design

## Stage 1: REST API Design

### 1. Get Notifications
GET /api/notifications

Headers:
Authorization: Bearer <token>

Response:
[
  {
    "id": "n1",
    "title": "Placement Drive",
    "message": "Company ABC is hiring",
    "type": "placement",
    "read": false,
    "createdAt": "2026-05-09T10:00:00Z"
  }
]

---

### 2. Mark Notification as Read
PATCH /api/notifications/:id/read

Response:
{
  "message": "Notification marked as read"
}

---

## Real-Time System (WebSockets)

Use Socket.io

Events:
- new_notification
- update_notification

Flow:
- Student connects to socket server
- Server pushes updates instantly
- Used for placements, events, results