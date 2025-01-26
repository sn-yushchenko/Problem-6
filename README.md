# Problem-6

**Overview**
------------

The Score Board System is designed to show the top 10 user scores on a website and update in real-time. Users complete actions that increase their scores, and upon completion, an API call is dispatched to the application server to update the score.

**System Requirements**
-----------------------

1.  **Website Frontend**:

    *   Displays a scoreboard showing the top 10 users.

    *   Updates the scoreboard dynamically (in real-time) when scores change.

    *   Must be responsive and capable of handling a large number of users.

2.  **Backend**:

    *   Manages user scores, ensuring integrity and preventing malicious score manipulation.

    *   Updates the scoreboard in real-time and allows for the processing of score updates.

3.  **Real-Time Communication**:

    *   The system needs a mechanism (such as WebSockets) to push real-time score updates to the frontend.


**API Design**
--------------

### 1\. **API Endpoint to Update Score**

*   **Endpoint**: POST /api/v1/update-score

*   **Description**: This API is triggered when a user completes an action that increases their score.

*   **Request**:

    *   json{ "user\_id": "12345", "score\_increment": 10}

*   **Response**:

    *   **Status Code**: 200 OK (if the update was successful)

    *   json{ "status": "success", "user\_id": "12345", "new\_score": 150}

    *   **Status Code**: 400 Bad Request (if the request was malformed or missing required fields)

    *   json{ "status": "error", "message": "Invalid request data"}

**Real-Time Updates**
---------------------

### **Implementation Approach**

To display live updates of the scoreboard to all connected users, we will implement a real-time communication mechanism. We will use the next approach:

1.  **WebSockets**: A WebSocket connection will be established between the client and the server. Once a user’s score is updated, the server will broadcast the updated score to all connected clients.


### **WebSocket Server**

*   **Protocol**: WebSocket

*   **Endpoint**: /ws/scoreboard

*   **Description**: The server will push updates to the connected clients when the scores are updated.

*   **Communication**:

    *   Client connects via WebSocket to the server and listens for updates.

    *   When a user’s score is updated, the server will broadcast the update to all connected clients.

    *   Each broadcast will contain the new top 10 user scores.

*   json{ "type": "score\_update", "top\_scores": \[ { "user\_id": "123", "username": "user1", "score": 150 }, { "user\_id": "456", "username": "user2", "score": 145 }, ... \]}


**Security Considerations**
---------------------------

### 1\. **Authentication and Authorization**

To prevent unauthorized users from manipulating scores, **JWT (JSON Web Tokens)** will be used for authentication. Only authenticated users with valid tokens will be allowed to update scores.

*   **JWT Validation**: The server will validate the JWT token sent in the Authorization header of the POST /api/v1/update-score request. If the token is invalid or missing, the request will be rejected with a 401 Unauthorized status.

*   **Role-Based Authorization**: Depending on your system's structure, users may have different roles (e.g., Admin, Regular User). Only users who have the necessary permissions should be allowed to update their scores. Admin users could have additional rights to manage scores, but regular users should only be able to update their own score.


### 2\. **Rate Limiting**

To prevent abuse (such as malicious users sending repeated requests to manipulate scores), **rate limiting** will be enforced on the /api/v1/update-score endpoint. Each user will be allowed a limited number of score updates within a specific time window (e.g., 10 updates per minute).

Rate-limiting headers will be included in the response:

*   **X-RateLimit-Limit**: The maximum number of allowed requests.

*   **X-RateLimit-Remaining**: The remaining number of requests for the current time window.

*   **X-RateLimit-Reset**: The time at which the rate limit will reset.


If a user exceeds the rate limit, they will receive a 429 Too Many Requests response.

### 3\. **Input Validation**

Input parameters like user\_id and score\_increment should be validated to ensure that:

*   user\_id is a valid user in the system.

*   score\_increment is a positive integer (non-negative).


Additionally, we can perform checks to prevent malicious users from trying to manipulate scores by sending extreme values or unauthorized user IDs.

**User Flow**
-------------

1.  **User Completes Action**: A user performs an action (e.g., a game level completed, an achievement unlocked).

2.  **Score Update API Call**: Upon completing the action, the frontend makes a POST /api/v1/update-score API request, passing the user’s ID and the score increment.

3.  **Backend Updates the Score**: The backend verifies the user’s identity and authorization, updates the user’s score in the database, and broadcasts the updated top 10 scores to all connected clients via WebSockets.

4.  **Frontend Updates Scoreboard**: All users viewing the scoreboard receive real-time updates and see the new top 10 scores.


**Error Handling**
------------------

*   **Invalid Request**: If the request data is malformed or missing required fields, return 400 Bad Request.

*   **Unauthorized Access**: If the user is not authenticated or the JWT token is invalid, return 401 Unauthorized.

*   **Forbidden Action**: If the user attempts to update a score that they do not have permission to modify, return 403 Forbidden.

*   **Internal Server Error**: In case of any unexpected failure in processing the request, return 500 Internal Server Error.


**Testing**
-----------

### 1\. **Unit Tests**

*   **Test for Valid Score Update**: Ensure that the score is correctly updated when the API receives a valid request.

*   **Test for Invalid Token**: Ensure that the API responds with 401 Unauthorized for requests with an invalid or missing token.

*   **Test for Rate Limiting**: Verify that the rate limiting mechanism works as expected and users are correctly throttled after the rate limit is exceeded.


### 2\. **Integration Tests**

*   **End-to-End Test**: Test the entire flow from user action to API request to WebSocket broadcast and real-time update on the frontend.


### 3\. **Load Testing**

*   **Simulate High Traffic**: Test how the system behaves under heavy load, particularly how the API and WebSocket connections handle a large number of users simultaneously updating their scores.


**Deployment**
--------------

1.  **WebSocket Server**: Deploy the WebSocket server alongside the main application server. Use a load balancer to distribute WebSocket connections evenly.

2.  **Database**: Ensure the database is optimized to handle frequent updates to the user scores.

3.  **Rate Limiting and Authentication Services**: These should be deployed as microservices or integrated into the main server depending on architecture.


## Contact

Serhii Yushchenko

Email: <sn-yushchenko@ukr.net>

Project Link: https://github.com/sn-yushchenko/Problem-6