# ❤️ Favorites

Zynvo allows users to save other users to their favorites list for quick access later.

The favorite relationship is **one-way**.

For example:

```text
User A → favorites → User B
````

does not automatically mean:

```text
User B → favorites → User A
```

Favorites are stored separately from the main user profile.

---

## 🎯 Purpose

The Favorites feature allows users to:

* Add another user to favorites
* View their favorite users
* Remove a user from favorites
* Check whether a user is already favorited

Favorites are associated with the authenticated user.

---

# 🗄️ Database

Favorites are stored in the:

```text
user_favorites
```

table.

### Structure

```text
user_favorites
│
├── id
├── user_id
├── favorite_user_id
├── created_at
└── updated_at
```

### Relationship

```text
users
  │
  │ user_id
  ▼
user_favorites
  │
  │ favorite_user_id
  ▼
users
```

For example:

```text
user_id = 32
favorite_user_id = 9
```

means:

```text
User 32 has favorited User 9
```

---

# 🔗 API Endpoints

All Favorites APIs require Firebase authentication.

Base URL:

```text
/api/v1
```

### Get Favorites

```http
GET /api/v1/users/favorites
```

### Add Favorite

```http
POST /api/v1/users/{user}/favorite
```

### Remove Favorite

```http
DELETE /api/v1/users/{user}/favorite
```

---

# 🔐 Authentication

The APIs are protected by the Firebase authentication middleware.

Request header:

```http
Authorization: Bearer FIREBASE_ID_TOKEN
Accept: application/json
```

The authenticated user is determined from the Firebase authentication middleware.

The mobile application does not send the authenticated user's ID for these operations.

---

# ❤️ Add Favorite

### Endpoint

```http
POST /api/v1/users/{user}/favorite
```

Example:

```http
POST /api/v1/users/9/favorite
```

If the authenticated user is:

```text
User ID = 32
```

the operation creates:

```text
user_id = 32
favorite_user_id = 9
```

---

## Request

No request body is required.

```http
Authorization: Bearer FIREBASE_ID_TOKEN
Accept: application/json
```

---

## Successful Response

```json
{
    "status": true,
    "message": "User added to favorites.",
    "data": {
        "user_id": 9
    }
}
```

HTTP status:

```text
201 Created
```

---

# 🚫 Favorite Yourself

A user cannot favorite their own account.

Example:

```text
Authenticated user = 32
Target user = 32
```

Response:

```json
{
    "status": false,
    "message": "You cannot favorite yourself.",
    "data": null
}
```

HTTP status:

```text
422 Unprocessable Entity
```

---

# ⚠️ Duplicate Favorite

If the user has already favorited another user, the API does not create another record.

Example:

```text
User 32 → User 9
```

already exists in:

```text
user_favorites
```

Trying to add User 9 again returns:

```json
{
    "status": false,
    "message": "User is already in your favorites.",
    "data": null
}
```

HTTP status:

```text
409 Conflict
```

---

# 📋 Get Favorites

### Endpoint

```http
GET /api/v1/users/favorites
```

This returns the users favorited by the authenticated user.

Example:

```text
Authenticated user = 32
```

Database:

```text
user_id | favorite_user_id
--------|-----------------
32      | 9
32      | 15
32      | 21
```

The API returns:

```text
User 9
User 15
User 21
```

---

## Successful Response

Example:

```json
{
    "status": true,
    "message": "Favorite users retrieved successfully.",
    "data": [
        {
            "id": 9,
            "uid": "xxxxxxxx",
            "name": "Anonymous",
            "gender": "female",
            "status": true
        },
        {
            "id": 15,
            "uid": "xxxxxxxx",
            "name": "Anonymous",
            "gender": "male",
            "status": true
        }
    ]
}
```

The public application should only expose information appropriate for the Zynvo profile.

Private account information such as the user's real Google identity should not be exposed through the Favorites API.

---

# ❌ Remove Favorite

### Endpoint

```http
DELETE /api/v1/users/{user}/favorite
```

Example:

```http
DELETE /api/v1/users/9/favorite
```

If the authenticated user is:

```text
User 32
```

the API removes:

```text
32 → 9
```

from `user_favorites`.

---

## Successful Response

```json
{
    "status": true,
    "message": "User removed from favorites.",
    "data": {
        "user_id": 9
    }
}
```

HTTP status:

```text
200 OK
```

---

# ⚠️ Removing a Non-Favorite User

If the user is not currently in the authenticated user's favorites:

```json
{
    "status": false,
    "message": "User is not in your favorites.",
    "data": null
}
```

HTTP status:

```text
404 Not Found
```

---

# 🔄 Favorite Flow

```text
                 Discover User
                       │
                       ▼
                ┌─────────────┐
                │ User Profile│
                └──────┬──────┘
                       │
                       ▼
                 ❤️ Favorite
                       │
                       ▼
                user_favorites
                       │
                       ▼
              Favorites List
                       │
              ┌────────┴────────┐
              ▼                 ▼
         View Profile      Remove Favorite
                                  │
                                  ▼
                           Delete relationship
```

---

# 🧩 Application Architecture

The Favorites feature follows the Laravel service architecture.

```text
Flutter App
     │
     ▼
FavoriteController
     │
     ▼
FavoriteService
     │
     ▼
MySQL
     │
     ▼
user_favorites
```

### Controller

Responsible for:

* Reading the authenticated user
* Validating authentication
* Passing the request to the service

### Service

Responsible for:

* Preventing self-favorites
* Checking target users
* Checking duplicate favorites
* Creating favorite relationships
* Removing favorites
* Retrieving favorite users

### Model

The `Favorite` model represents the:

```text
user_favorites
```

database table.

---

# 🧱 Favorite Model

The model uses:

```php
protected $table = 'user_favorites';
```

Fillable fields:

```php
protected $fillable = [
    'user_id',
    'favorite_user_id',
];
```

The model provides two relationships:

```php
public function user(): BelongsTo
{
    return $this->belongsTo(
        User::class,
        'user_id'
    );
}
```

and:

```php
public function favoriteUser(): BelongsTo
{
    return $this->belongsTo(
        User::class,
        'favorite_user_id'
    );
}
```

---

# 🔗 User Relationship

The `User` model can expose users that the authenticated user has favorited.

Conceptually:

```text
User
 │
 └── favoriteUsers
          │
          ├── User
          ├── User
          └── User
```

The relationship represents:

```text
authenticated user
        │
        ▼
favorite relationships
        │
        ▼
favorite users
```

---

# 🔐 Privacy

Favorites do not change the target user's public profile visibility.

Adding someone to favorites does not automatically:

* Reveal their real name
* Reveal their private photo
* Reveal their email
* Reveal their phone number
* Change their profile privacy

The application continues to use the user's public Zynvo profile information.

---

# 🚫 Block Interaction

Blocking and favoriting are separate relationships.

For example:

```text
User 32
   │
   ├── Favorite → User 9
   │
   └── Block → User 15
```

A block should prevent normal interaction with the blocked user even if a previous favorite relationship existed.

The application should therefore prioritize blocking over normal social interactions.

---

# 🔍 Discover Interaction

Favorites are separate from the Discover matching system.

A user can:

```text
Discover
   ↓
View User
   ↓
Favorite User
```

The user can later access that person from:

```text
Favorites
```

Favorites do not automatically mean that the user should appear in Discover again after being skipped or excluded by another discovery rule.

---

# 📱 Flutter Integration

The Flutter application can use the APIs as follows:

### Add

```text
POST /api/v1/users/{userId}/favorite
```

### List

```text
GET /api/v1/users/favorites
```

### Remove

```text
DELETE /api/v1/users/{userId}/favorite
```

The Flutter application should update the favorite icon based on the API response.

Example:

```text
♡  →  ❤️
```

when a user is successfully added to favorites.

---

# 🧪 API Testing

### Add

```http
POST /api/v1/users/9/favorite
```

Expected:

```json
{
    "status": true,
    "message": "User added to favorites."
}
```

### List

```http
GET /api/v1/users/favorites
```

### Remove

```http
DELETE /api/v1/users/9/favorite
```

Expected:

```json
{
    "status": true,
    "message": "User removed from favorites."
}
```

---

# 📊 Summary

The Zynvo Favorites system provides a simple one-way relationship between users.

```text
User A
  │
  │ ❤️
  ▼
User B
```

The relationship is stored independently in:

```text
user_favorites
```

and can be created, retrieved, and removed through authenticated REST APIs.

### Implemented APIs

| Method   | Endpoint                 | Purpose            |
| -------- | ------------------------ | ------------------ |
| `GET`    | `/users/favorites`       | Get favorite users |
| `POST`   | `/users/{user}/favorite` | Add favorite       |
| `DELETE` | `/users/{user}/favorite` | Remove favorite    |

---

## ✅ Status

**Implemented**

The Favorites functionality is implemented as part of the Zynvo Laravel REST API and integrated with the anonymous social discovery experience.