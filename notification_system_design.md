# Notification System Design

---

# Stage 1 – REST API Design

## Objective
Design a scalable notification system that supports creating, fetching, updating, deleting, and real-time delivery of notifications.

---

## Core Features
- Create notifications
- Fetch user notifications
- Mark notification as read
- Mark all notifications as read
- Delete notifications
- Real-time notification delivery

---

## Base URL
/api/v1/notifications

---

## 1. Create Notification API

### Endpoint
POST /api/v1/notifications

### Description
Creates and sends notifications to one or more users.

### Request Body
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

### Response
{
  "success": true,
  "notificationId": "n1001",
  "message": "Notification created successfully"
}

---

## 2. Get Notifications API

GET /api/v1/notifications?page=1&limit=20&status=UNREAD

### Response
{
  "success": true,
  "notifications": [
    {
      "notificationId": "n1001",
      "title": "Appointment Confirmed",
      "message": "Your appointment is confirmed",
      "status": "UNREAD",
      "createdAt": "2026-05-09T10:00:00Z"
    }
  ]
}

---

## 3. Mark as Read API
PATCH /api/v1/notifications/{notificationId}/read

---

## 4. Mark All as Read API
PATCH /api/v1/notifications/read-all

---

## 5. Delete Notification API
DELETE /api/v1/notifications/{notificationId}

---

## Notification Schema
{
  "notificationId": "string",
  "userId": "string",
  "title": "string",
  "message": "string",
  "type": "SYSTEM | ALERT | PROMOTIONAL",
  "priority": "LOW | MEDIUM | HIGH",
  "status": "READ | UNREAD",
  "metadata": {},
  "createdAt": "timestamp"
}

---

## Real-Time Notification System

### Technology Used
- WebSockets (Socket.IO)
- Redis Pub/Sub

### Flow
Backend → Redis → WebSocket Server → Client

---

# Stage 2 – Database Design & Scalability

---

## 1. Database Choice

### PostgreSQL (Primary DB)
### Redis (Cache + Real-time)

---

## Why PostgreSQL
- ACID compliance
- Strong relational model
- JSON support
- Indexing support
- Scalable with replication

---

## Why Redis
- Real-time notifications (Pub/Sub)
- Caching unread counts
- Reduces DB load
- Fast performance

---

## 2. Database Schema

### Users Table
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW()
);

---

### Notifications Table
CREATE TABLE notifications (
    notification_id UUID PRIMARY KEY,
    title VARCHAR(255),
    message TEXT,
    type VARCHAR(50),
    priority VARCHAR(20),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

---

### User Notifications Table
CREATE TABLE user_notifications (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(user_id),
    notification_id UUID REFERENCES notifications(notification_id),
    status VARCHAR(20) DEFAULT 'UNREAD',
    is_deleted BOOLEAN DEFAULT FALSE,
    read_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

---

### Notification Preferences Table
CREATE TABLE notification_preferences (
    user_id UUID PRIMARY KEY,
    email BOOLEAN DEFAULT TRUE,
    sms BOOLEAN DEFAULT FALSE,
    push BOOLEAN DEFAULT TRUE,
    system BOOLEAN DEFAULT TRUE
);

---

## 3. Indexing Strategy
CREATE INDEX idx_user_notifications_user_id ON user_notifications(user_id);
CREATE INDEX idx_user_notifications_status ON user_notifications(status);
CREATE INDEX idx_notifications_created_at ON notifications(created_at DESC);

---

## 4. Scalability Problems & Solutions

### Problem 1: Slow Queries
Solution: Indexing + pagination

### Problem 2: High Load
Solution: Read replicas

### Problem 3: Real-time overload
Solution: Redis Pub/Sub + Kafka

### Problem 4: Storage growth
Solution: Archiving old notifications

---

## 5. SQL Queries

### Create Notification
INSERT INTO notifications VALUES (
    gen_random_uuid(),
    'Appointment Confirmed',
    'Your appointment is confirmed',
    'SYSTEM',
    'HIGH',
    '{"appointmentId":"APT1001"}',
    NOW()
);

---

### Assign Notification to User
INSERT INTO user_notifications VALUES (
    gen_random_uuid(),
    'USER_ID',
    'NOTIFICATION_ID',
    'UNREAD',
    FALSE,
    NULL,
    NOW()
);

---

### Fetch Notifications
SELECT n.notification_id, n.title, n.message, un.status
FROM user_notifications un
JOIN notifications n ON n.notification_id = un.notification_id
WHERE un.user_id = 'USER_ID'
AND un.is_deleted = FALSE
ORDER BY n.created_at DESC;

---

### Mark as Read
UPDATE user_notifications
SET status = 'READ', read_at = NOW()
WHERE notification_id = 'NOTIFICATION_ID';

---

### Delete Notification
UPDATE user_notifications
SET is_deleted = TRUE
WHERE notification_id = 'NOTIFICATION_ID';

---

## Final Summary

This system provides:
- Scalable REST APIs
- Reliable database design
- Real-time notification delivery
- Efficient querying using indexing
- High scalability using Redis and PostgreSQL