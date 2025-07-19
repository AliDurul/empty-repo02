# Foundation Microservices - Blog Platform

A full-stack microservices-based blog platform demonstrating event-driven architecture, service communication, and containerization. This project implements a simple blog system where users can create posts and add comments with moderation capabilities.

## 🏗️ Architecture Overview

This project follows a microservices architecture with the following services:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │    Posts    │    │  Comments   │
│  (React)    │    │  Service    │    │   Service   │
│   :3000     │    │   :4000     │    │    :4001    │
└─────┬───────┘    └──────┬──────┘    └──────┬──────┘
      │                   │                  │
      │                   │                  │
      └───────────────────┼──────────────────┘
                          │
                ┌─────────▼──────────┐
                │    Event Bus       │
                │    Service         │
                │      :4005         │
                └─────────┬──────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼──────┐ ┌────────▼────────┐ ┌──────▼──────┐
│  Moderation  │ │     Query       │ │   Future    │
│   Service    │ │    Service      │ │  Services   │
│    :4003     │ │     :4002       │ │             │
└──────────────┘ └─────────────────┘ └─────────────┘
```

## 🚀 Services Description

### 1. **Posts Service** (Port 4000)
- **Purpose**: Manages blog posts creation and storage
- **Responsibilities**:
  - Create new posts
  - Store posts in memory
  - Emit `PostCreated` events to the event bus
- **Endpoints**:
  - `GET /posts` - Retrieve all posts
  - `POST /posts` - Create a new post

### 2. **Comments Service** (Port 4001)
- **Purpose**: Handles comments for posts
- **Responsibilities**:
  - Create comments for specific posts
  - Store comments with moderation status
  - Handle comment status updates from moderation
  - Emit `CommentCreated` and `CommentUpdated` events
- **Endpoints**:
  - `GET /posts/:id/comments` - Get comments for a post
  - `POST /posts/:id/comments` - Create a comment

### 3. **Event Bus Service** (Port 4005)
- **Purpose**: Central communication hub for all services
- **Responsibilities**:
  - Receive events from all services
  - Broadcast events to all registered services
  - Store event history for replay capabilities
- **Endpoints**:
  - `POST /events` - Receive and broadcast events
  - `GET /events` - Retrieve event history

### 4. **Moderation Service** (Port 4003)
- **Purpose**: Automated content moderation
- **Responsibilities**:
  - Review comments for inappropriate content
  - Automatically reject comments containing "orange"
  - Approve all other comments
  - Emit `CommentModerated` events
- **Events Handled**: `CommentCreated`

### 5. **Query Service** (Port 4002)
- **Purpose**: Data aggregation and read optimization
- **Responsibilities**:
  - Maintain materialized view of posts with their comments
  - Sync with event history on startup
  - Provide fast read access to aggregated data
- **Endpoints**:
  - `GET /posts` - Get all posts with their comments
- **Events Handled**: `PostCreated`, `CommentCreated`, `CommentUpdated`

### 6. **Client Service** (Port 3000)
- **Purpose**: React-based frontend application
- **Features**:
  - Create new posts
  - View all posts
  - Add comments to posts
  - Real-time display of comment moderation status

## 🛠️ Technology Stack

- **Backend**: Node.js, Express.js
- **Frontend**: React.js
- **HTTP Client**: Axios
- **Containerization**: Docker
- **Development**: Nodemon for hot-reloading

## 📋 Prerequisites

- Node.js (v14 or higher)
- npm or yarn
- Docker (optional, for containerization)

## 🚦 Getting Started

### Running with Node.js

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd Foundation-Microservices
   ```

2. **Install dependencies for all services**
   ```bash
   # Posts service
   cd posts && npm install && cd ..
   
   # Comments service
   cd comments && npm install && cd ..
   
   # Event Bus service
   cd event-bus && npm install && cd ..
   
   # Moderation service
   cd moderation && npm install && cd ..
   
   # Query service
   cd query && npm install && cd ..
   
   # Client
   cd client && npm install && cd ..
   ```

3. **Start all services** (in separate terminals)
   ```bash
   # Terminal 1 - Event Bus (start this first)
   cd event-bus && npm start
   
   # Terminal 2 - Posts Service
   cd posts && npm start
   
   # Terminal 3 - Comments Service
   cd comments && npm start
   
   # Terminal 4 - Query Service
   cd query && npm start
   
   # Terminal 5 - Moderation Service
   cd moderation && npm start
   
   # Terminal 6 - Client
   cd client && npm start
   ```

### Running with Docker

Each service includes a `Dockerfile` for containerization:

```bash
# Build and run a service (example for posts)
cd posts
docker build -t posts-service .
docker run -p 4000:4000 posts-service
```

## 🎯 Event Flow

### Creating a Post
1. User creates post via React client
2. Client sends POST request to Posts Service
3. Posts Service stores post and emits `PostCreated` event
4. Event Bus broadcasts event to all services
5. Query Service receives event and updates materialized view

### Creating a Comment
1. User adds comment via React client
2. Client sends POST request to Comments Service
3. Comments Service stores comment with "pending" status
4. Comments Service emits `CommentCreated` event
5. Event Bus broadcasts to all services
6. Moderation Service processes the comment
7. Moderation Service emits `CommentModerated` event
8. Comments Service updates comment status
9. Comments Service emits `CommentUpdated` event
10. Query Service updates materialized view with final comment status

## 🔧 Service Ports

| Service | Port | Purpose |
|---------|------|---------|
| Client | 3000 | React frontend |
| Posts | 4000 | Post management |
| Comments | 4001 | Comment management |
| Query | 4002 | Data aggregation |
| Moderation | 4003 | Content moderation |
| Event Bus | 4005 | Event broadcasting |

## 🎨 Key Features

- **Event-Driven Architecture**: Services communicate through events
- **Data Consistency**: Eventually consistent through event sourcing
- **Resilience**: Services can recover by replaying events
- **Scalability**: Services can be scaled independently
- **Fault Tolerance**: Services continue operating if others fail

## 🧪 Testing the Application

1. Open http://localhost:3000 in your browser
2. Create a new post using the form
3. Add comments to the post
4. Try adding a comment with the word "orange" to see moderation in action
5. Observe how comments are automatically approved or rejected

## 📚 Learning Objectives

This project demonstrates:
- Microservices architecture patterns
- Event-driven communication
- Service decoupling
- Data consistency in distributed systems
- Event sourcing concepts
- Container-based deployment

## 🔮 Future Enhancements

- Add authentication and user management
- Implement persistent storage (database)
- Add API gateway
- Implement service discovery
- Add monitoring and logging
- Deploy to cloud platforms (AWS, GCP, Azure)

## 📄 License

This project is for educational purposes and learning microservices architecture.