# API Service Module Specification

## Overview

This document specifies the design and functionality of the API Service module. The module will handle requests for updating user scores, retrieving the top scores, and ensuring real-time updates for the scoreboard. Additionally, it includes measures to prevent unauthorized score manipulation.

---

## Features

1. **Score Update API**
   - Allows users to update their scores upon completing an action.
2. **Leaderboard Retrieval API**
   - Provides the top user scores.
3. **Real-time Score Updates**
   - Uses WebSocket to broadcast changes to the leaderboard to all connected clients.
4. **Security Measures**
   - Implements mechanisms to prevent unauthorized score updates and detect abnormal behavior.

---

## API Endpoints

### 1. Update Score

- **Endpoint:** `/api/scoreboards`
- **Method:** `PATCH`
- **Authorization:**
  - Required. Clients must include a valid bearer token in the `Authorization` header:  
    `Authorization: Bearer <token>`
- **Request Body:**

  ```json
  {
    "actionId": "string"
  }
  ```

- **Response**:

  ```json
  {
    "success": true,
    "message": "Score updated successfully."
  }
  ```

- **Notes**:
  - Only authenticated users can update scores.
  - The system tracks each `actionId` to ensure that each action is used only once.
  - The API validates the `actionId` to ensure it hasn’t been used before.
  - Upon validation, the user’s score is updated in the database.
  - In case of an invalid token or repeated `actionId`, the API should return an appropriate error (e.g., 401 Unauthorized or 400 Bad Request).

### 2. Retrieve Leaderboard

- **Endpoint**: `/api/scoreboards`
- **Method**: GET
- **Query Parameters**:
  - `limit`: Number of top scores to retrieve. Defaults to 10.
- **Response**:

  ```json
  {
    "success": true,
    "data": [
      { "userId": "string", "score": 80 },
      { "userId": "string", "score": 55 }
    ]
  }
  ```

### 3. Real-time Updates (WebSocket)

- **Endpoint**: `/ws/scoreboards`

- **Functionality**:

  - Establishes a WebSocket connection to broadcast score changes.
  - Only the relevant updated user data is sent to reduce bandwidth.

- **Authentication**:

  - Required at connection time (e.g., token validation during the WebSocket handshake).

---

## Database Schema

### Users Table
```
| Column   | Type    | Constraints | Notes  |
| -------- | ------- | ----------- | ------ |
| id       | UUID    | Primary Key |        |
| name     | String  |             |        |
| username | String  | Unique      |        |
| password | String  |             | Hashed |
| score    | Integer | Default = 0 |        |

### Actions Table

| Column      | Type   | Constraints | Notes |
| ----------- | ------ | ----------- | ----- |
| id          | UUID   | Primary Key |       |
| description | String |             |       |

### User Action table (Base on business case)

| Column      | Type   | Constraints | Notes |
| ----------- | ------ | ----------- | ----- |
| id          | UUID   | Primary Key |       |
| userId      | UUID   |             |       |
| actionId    | UUID   |             |       |
| description | String |             |       |

```

## Flow of Execution

```
graph TD
    A[User completes an action] --> B[Frontend sends PATCH /api/scoreboards]
    B --> C[API validates user authentication (JWT)]
    C --> D[API verifies that the actionId is unique and valid]
    D --> E[Update user score in the database]
    E --> F[Broadcast updated leaderboard via WebSocket (/ws/scoreboards)]
    F --> G[Clients receive real-time update and refresh the displayed leaderboard]

```

## Improvements Suggestions

1. **Caching for Leaderboard**:
   - Use an in-memory datastore (e.g., Redis) to cache the top scores and reduce database load for frequent GET requests.
2. **Rate Limiting**:
   - Implement rate limiting on the PATCH /api/scoreboards endpoint to prevent abuse from automated or malicious requests.
3. **Audit Logging**:
   - Log all score update requests (both successful and failed) for monitoring, debugging, and audit purposes.
4. **Action Verification**:
   - Add a server-side mechanism to validate the authenticity of `actionId` before updating scores. (Base on business case)
5. **WebSocket Scaling**:
   - Consider using a message broker (e.g., Redis Pub/Sub or Kafka) to manage and scale real-time WebSocket connections for a large number of clients.
6. **Endpoint Separation:**:
   = Consider separating update and retrieval endpoints into distinct URLs (e.g., /api/score/update and /api/scoreboard) for better clarity and RESTful design.

---

## Technology Stack

- **Backend Framework**: Express.js
- **Database**: PostgreSQL
- **Real-time Communication**: Socket.IO
- **Authentication**: JWT-based authentication

---

## Deliverables

1. Fully functional APIs as per the specification.
2. Real-time WebSocket integration.
3. Database schema migrations.
4. Unit and integration tests for all features.
5. Documentation and setup instructions.

---

## Notes for Developers

- Ensure the module is extensible to add more actions in the future.
- Adhere to clean code practices and include meaningful comments.
- Test the APIs under high concurrency to ensure stability.
- Keep documentation up-to-date with any changes or new features.

---
