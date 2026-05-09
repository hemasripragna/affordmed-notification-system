# Notification System Design

---

# Stage 1 – REST API Design

## Objective
Design REST APIs for a scalable notification system supporting create, fetch, update, delete, and real-time notifications.

---

## Base URL
/api/v1/notifications

---

## 1. Create Notification
POST /api/v1/notifications

Request:
{
  "recipientIds": ["u101", "u102"],
  "title": "Appointment Confirmed",
  "message": "Your appointment is confirmed.",
  "type": "SYSTEM",
  "priority": "HIGH",
  "metadata": {
    "appointmentId": "APT1001"
  }
}

Response:
{
  "success": true,
  "notificationId": "n1001"
}

---

## 2. Get Notifications
GET /api/v1/notifications?page=1&limit=20

Response:
{
  "notifications": []
}

---

## 3. Mark as Read
PATCH /api/v1/notifications/{id}/read

---

## 4. Mark All as Read
PATCH /api/v1/notifications/read-all

---

## 5. Delete Notification
DELETE /api/v1/notifications/{id}

---

## Real-time System
WebSocket + Redis Pub/Sub
Backend → Redis → WebSocket → Client

---

# Stage 2 – DB Design & Scalability

## Database Choice
- PostgreSQL (primary)
- Redis (cache + real-time)

---

## Tables

Users:
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  name TEXT,
  email TEXT
);

Notifications:
CREATE TABLE notifications (
  notification_id UUID PRIMARY KEY,
  title TEXT,
  message TEXT,
  type TEXT,
  priority TEXT,
  metadata JSONB,
  created_at TIMESTAMP
);

User Notifications:
CREATE TABLE user_notifications (
  id UUID PRIMARY KEY,
  user_id UUID,
  notification_id UUID,
  status TEXT DEFAULT 'UNREAD',
  is_deleted BOOLEAN DEFAULT FALSE
);

---

## Indexes
CREATE INDEX idx_user ON user_notifications(user_id);
CREATE INDEX idx_status ON user_notifications(status);

---

## Scalability
- Read replicas
- Partitioning
- Redis caching
- Kafka queues

---

# Stage 3 – Query Optimization

## Problem Query
SELECT * FROM notifications
WHERE student_id=1042 AND isRead=false
ORDER BY createdAt;

---

## Issue
- Full table scan
- No index
- Sorting overhead

---

## Fix
CREATE INDEX idx_student_unread
ON notifications(student_id, isRead, createdAt);

---

## Optimized Query
SELECT * FROM notifications
WHERE student_id=1042 AND isRead=false;

---

## Placement Query (7 days)
SELECT * FROM notifications
WHERE type='Placement'
AND createdAt >= NOW() - INTERVAL 7 DAY;

---

# Stage 4 – Performance Tuning

## Problem
DB overload due to frequent reads

---

## Solutions

### 1. Redis Cache
Store frequent notifications

### 2. Pagination
LIMIT 20 OFFSET 0;

### 3. Event Driven System
Kafka / Queue based processing

---

## Tradeoffs
- Cache: fast but stale
- Queue: scalable but complex

---

# Stage 5 – Bulk Notification System

## Problem
50,000 users notified via loop → slow

---

## Bad Approach
for user in users:
  sendEmail()
  saveDB()

---

## Improved Approach
- Use Kafka/RabbitMQ
- Batch processing (1000 users)
- Parallel workers

---

## Flow
HR → Queue → Workers → Email + DB + Push

---

## Benefits
- Scalable
- Fast
- Fault tolerant

---

# Stage 6 – API Optimization

## Problem
Large JSON responses slow API

---

## Solutions
- Pagination
- Filtering by type
- Select only required fields

---

## Optimized Response
{
  "id": "",
  "type": "",
  "message": "",
  "timestamp": ""
}

---

# Stage 7 – Frontend (React)

## Setup
npx create-react-app notification-ui
cd notification-ui
npm install axios socket.io-client

---

## Fetch API
axios.get("/api/v1/notifications")

---

## Real-time updates
socket.on("notification", updateUI);

---

## UI Structure
- NotificationList
- NotificationItem
- Dashboard

---

## Flow
React UI → API Gateway → Backend → DB + Redis → WebSocket → UI update

---

# FINAL SUMMARY

- REST API design implemented
- Scalable DB architecture using PostgreSQL + Redis
- Optimized queries with indexing
- Performance improved using caching and queues
- Bulk notifications handled via batch processing
- Frontend built using React with real-time updates