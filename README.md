# Microservices Blog App

This is a simple blog application built using a microservices architecture. It demonstrates the communication between services using an **Event Bus** and handles asynchronous data consistency.

## Architecture

The application is broken down into the following independent services:

| Service | Port | Description |
| :--- | :--- | :--- |
| **Posts** | `4000` | Handles creation and storage of blog posts. |
| **Comments** | `4001` | Handles creation and storage of comments. Associates comments with posts. |
| **Query** | `4002` | An optimized query service that aggregates posts and comments into a single data structure for efficient reading. Solving the N+1 problem. |
| **Moderation** | `4003` | Watcher service that automatically moderates comments. It rejects any comment containing the word **"orange"**. |
| **Event Bus** | `4005` | Receives events from services and broadcasts them to all other services. Also stores a history of events for data synchronization (e.g., when a service comes back online). |
| **Client** | `3000` | A React frontend application for users to interact with the blog. |

## Data Flow & Features

### 1. Creating a Post
- User submits a form on the Client.
- Request goes to **Posts Service** (`POST /posts`).
- **Posts Service** emits `PostCreated` event to **Event Bus**.
- **Event Bus** broadcasts to **Query Service**.

### 2. Creating a Comment
- User submits a comment on a post.
- Request goes to **Comments Service** (`POST /posts/:id/comments`).
- Comment is created with status **'pending'**.
- **Comments Service** emits `CommentCreated` event.
- **Event Bus** broadcasts to **Query** and **Moderation**.

### 3. Moderation Process
- **Moderation Service** receives `CommentCreated`.
- It checks if the content contains the word "orange".
- If it does, status becomes **'rejected'**, otherwise **'approved'**.
- Emits `CommentModerated` event.
- **Comments Service** receives this, updates its database, and emits `CommentUpdated`.
- **Query Service** receives update and reflects the new status to the user.

## How to Run

You will need to run each service in a separate terminal window.

```bash
# 1. Posts Service
cd posts
npm start

# 2. Comments Service
cd comments
npm start

# 3. Query Service
cd query
npm start

# 4. Moderation Service
cd moderation
npm start

# 5. Event Bus
cd event-bus
npm start

# 6. React Client
cd client
npm start
```

Open [http://localhost:3000](http://localhost:3000) to view the app.
