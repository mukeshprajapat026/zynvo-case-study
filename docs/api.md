# Zynvo API Documentation

## Base URL

`https://zynvo-api.syslotech.com`

## API Version

`/v1`

---

## Authentication

All protected user APIs require:

```text
Authorization: Bearer {firebase_id_token}


1. Health
GET /health
Example:
GET https://zynvo-api.syslotech.com/health
Response:
{
    "status": true,
    "message": "Zynvo API is healthy.",
    "data": {
        "service": "Zynvo API",
        "database": "connected"
    }
}

2. User
POST /v1/user/sync
Authentication
Firebase
Request
{
    "name": "Test User",
    "age": 28,
    "gender": "male"
}
Response
{
    "status": true,
    "message": "User synced successfully.",
    "data": {}
}
Save Device Token
POST /v1/user/device-token

3. Discovery
GET /v1/discover
Authentication
Firebase
Example
GET /v1/discover

4. Chat Rooms
GET /v1/chat-rooms

Create Chat Room
POST /v1/chat-rooms
Request
{
    "receiver_id": 25
}

5. Messages
GET /v1/chat-rooms/{chatRoom}/messages

Send Message
POST /v1/chat-rooms/{chatRoom}/messages
Request
{
    "type": "text",
    "message": "Hello!"
}

6. Media
POST /v1/chat-rooms/{chatRoom}/media
Content-Type
multipart/form-data

7. Block
GET /v1/users/blocked

Block User
POST /v1/users/{user}/block

Unblock User 
DELETE /v1/users/{user}/block

8. Favorites
GET /v1/users/favorites

Favorite User
POST /v1/users/{user}/favorite

Remove Favorite
DELETE /v1/users/{user}/favorite

9. Reports
POST /v1/users/{user}/report
Request
{
    "reason": "inappropriate_content",
    "description": "Inappropriate messages."
}
Available Reasons
spam
harassment
inappropriate_content
fake_profile
adult_content
hate_speech
other

10. Admin
Dashboard
GET /v1/admin/dashboard

Get Users
GET /v1/admin/users

Get User
GET /v1/admin/users/{id}

Update User
PUT/PATCH /v1/admin/users/{id}

Error Responses
401 Unauthorized
{
    "status": false,
    "message": "Authentication failed. Please login again.",
    "data": null
}
404 Not Found
{
    "status": false,
    "message": "User not found.",
    "data": null
}
422 Validation Error
{
    "status": false,
    "message": "Validation failed.",
    "errors": {}
}

Postman Collection
The complete API collection is maintained in Postman and follows the same structure as this documentation.
### So your `api.md` will basically become your API reference

```text
docs/api.md
│
├── Authentication
├── Health
├── User
├── Discovery
├── Chat Rooms
├── Messages
├── Media
├── Block
├── Favorites
├── Reports
└── Admin