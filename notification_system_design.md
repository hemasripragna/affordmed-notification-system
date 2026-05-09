# Stage 1

# Notification System Design

## Objective
Design a scalable REST API-based Notification System that allows applications to:

- Send notifications to users
- Fetch notifications for logged-in users
- Mark notifications as read/unread
- Delete notifications
- Support real-time notifications

---

# Core Actions Supported

| Action | Description |
|---|---|
| Create Notification | Send notification to one or more users |
| Get Notifications | Fetch logged-in user's notifications |
| Mark as Read | Mark notification as read |
| Mark All as Read | Mark all notifications as read |
| Delete Notification | Delete notification |
| Real-Time Notification | Push notification instantly |

---

# Base URL

```http
/api/v1/notifications