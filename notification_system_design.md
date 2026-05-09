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
We choose MongoDB (NoSQL database) for the notification system.

Why MongoDB?
Flexible schema for different notification types
High scalability for large number of students
Fast read/write operations for real-time updates
Easy integration with Node.js backend
📊 Database Schema
Collection: notifications
{
  "_id": "ObjectId",
  "userId": "string",
  "title": "string",
  "message": "string",
  "type": "placement | event | result",
  "read": false,
  "createdAt": "date"
}
🔍 Indexing Strategy

To improve performance:

Create index on userId
Optional index on type for filtering
Helps in fast retrieval of user notifications
📡 Database Queries
1. Get Notifications for User
db.notifications.find({ userId: "123" })
2. Mark Notification as Read
db.notifications.updateOne(
  { _id: ObjectId("n1") },
  { $set: { read: true } }
)
3. Insert New Notification
db.notifications.insertOne({
  userId: "123",
  title: "Placement Drive",
  message: "Company ABC is hiring",
  type: "placement",
  read: false,
  createdAt: new Date()
})
📈 Handling Large Data (Scalability)
Use pagination (limit + skip) for large notification lists
Use sharding if data grows across multiple servers
Use TTL indexes to delete old notifications automatically
Cache frequently accessed notifications using Redis (optional)
🔄 Real-time Sync Support
Database updates trigger WebSocket events
New inserts → pushed instantly to users
Keeps UI always updated without refresh