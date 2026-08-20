# 💜 Zynvo

### Anonymous Social Chat Platform

**Meet. Chat. Connect. Anonymously.**

Zynvo is a privacy-focused social chat platform designed to help users discover and connect with new people while keeping their real identity private.

Users can discover compatible people based on age and gender preferences, create an anonymous public profile, and communicate through text, voice, and image-based conversations.

---

## 🚧 Project Status

**Active Development**

Zynvo is currently under active development. The backend APIs, database architecture, authentication, discovery, chat rooms, media handling, favorites, blocking, reporting, and admin functionality are being implemented incrementally.

The production application source code is maintained separately in a private repository.

---

# ✨ Features

## 👤 Anonymous Profiles

Zynvo separates private account information from information displayed inside the application.

### Private Account Information

The `users` table stores information such as:

- Real name
- Email
- Phone number
- Google authentication information
- Private profile photo
- Account status
- Authentication-related information

This information is not exposed as the user's public application profile.

### Public Application Profile

The application uses a separate `user_profiles` table for profile information displayed to other users.

Public profile information includes:

- Anonymous nickname
- Gender
- Age
- Looking-for preference
- Age range
- Online status
- Application photos

Example:
Private Name: Mukesh Kumar
Application Name: Anonymous

The real name remains private while the anonymous nickname is displayed in the application.

---

# 🔐 Privacy

Privacy is a core part of the Zynvo architecture.

The platform separates private account information from public application profile information.

                    Zynvo User
                        │
            ┌───────────┴───────────┐
            │                       │
            ▼                       ▼
     Private Account          Public Profile
            │                       │
            ▼                       ▼
         users                 user_profiles
            │                       │
            ├── real name           ├── nickname
            ├── email               ├── gender
            ├── phone               ├── age
            ├── Google photo        ├── looking_for
            └── private data        └── age preferences
                                        │
                                        ▼
                                  user_photos

### Private Google / Real Photo

The Google profile photo stored in the `users` table is treated as private.

Photos displayed inside the application are stored separately in the `user_photos` table.

This prevents the private Google profile image from automatically becoming the user's public application image.

---

# 🎯 User Discovery

Zynvo provides a discovery system that searches for compatible users based on the authenticated user's profile preferences.

The discovery API reads the matching preferences from the logged-in user's profile.

Example:

{
    "lookingFor": "female",
    "ageRangeMin": 18,
    "ageRangeMax": 40
}

These values are read from the authenticated user's profile rather than being required as query parameters.

### Discovery Flow

```text
Google Login
     │
     ▼
Load User Profile
     │
     ▼
Read Matching Preferences
     │
     ▼
Find Compatible Users
     │
     ├──────────────┐
     ▼              ▼
   Chat            Skip
     │
     ▼
Text / Voice / Image
     │
     ├────► Favorite
     ├────► Block
     └────► Report
```

Users who already have an established chat relationship can be excluded from the discovery list.

Blocked users can also be excluded from discovery.

---

# 💬 Chat

Zynvo uses a hybrid chat architecture.

## Text Messages

Text conversations are handled directly through Firebase.

```text
Flutter App
     │
     ▼
Firebase
     │
     ▼
Other Flutter User
```

Text messages are not duplicated in the Laravel MySQL database.

## Media Messages

Images and voice messages are handled through the Laravel API.

```text
Flutter App
     │
     ▼
Laravel API
     │
     ├── Private Storage
     │
     └── MySQL
```

The `messages` table stores metadata for uploaded media.

---

# 💬 Chat Rooms

Each conversation has a dedicated chat room.

The `chat_rooms` table contains:

```text
chat_rooms
├── id
├── user1_id
├── user2_id
├── room_status
├── blocked_by
├── last_message_id
├── last_message_at
├── created_at
└── updated_at
```

### Room Status

Supported room statuses include:

```text
active
blocked
deleted
```

### Create Chat Room

```http
POST /api/v1/chat-rooms
```

Example request:

```json
{
    "receiver_id": 9
}
```

The API uses the internal `users.id` value to create the relationship.

The backend can prevent duplicate chat rooms between the same two users.

### Get Chat Rooms

```http
GET /api/v1/chat-rooms
```

The response contains the rooms belonging to the authenticated user and the other user's public profile information.

---

# 🖼️ Image Sharing

Users can send images inside a chat room.

```text
Flutter
    │
    │ multipart/form-data
    ▼
POST /api/v1/chat-rooms/{chatRoom}/media
    │
    ▼
Laravel
    │
    ├── Validate file
    ├── Generate unique filename
    ├── Store private media
    └── Create message record
```

The media upload endpoint supports media fields such as:

```text
media[]
message_type
```

Each uploaded media file can create a separate message record.

---

# 🎤 Voice Messages

Voice messages are also uploaded through Laravel.

Example:

```text
message_type = voice
media[] = voice.m4a
voice_duration = 15
```

The database stores:

```text
message_type
file
voice_duration
sender_id
receiver_id
room_id
```

The actual media file is stored separately from the database.

---

# 🗄️ Messages

The `messages` table is used for chat-related media metadata and message information handled by the Laravel backend.

Structure:

```text
messages
├── id
├── room_id
├── sender_id
├── receiver_id
├── message_type
├── message
├── file
├── voice_duration
├── is_read
├── deleted_at
├── created_at
└── updated_at
```

Supported message types include:

```text
text
image
voice
```

Text messaging is handled through Firebase in the current architecture, while Laravel manages uploaded media and related database records.

---

# ❤️ Favorites

Users can add other users to their favorites.

### Add Favorite

```http
POST /api/v1/favorites/{user}
```

### Remove Favorite

```http
DELETE /api/v1/favorites/{user}
```

### Get Favorites

```http
GET /api/v1/favorites
```

Favorites are stored in:

```text
user_favorites
├── id
├── user_id
├── favorite_user_id
├── created_at
└── updated_at
```

The relationship is directional:

```text
User A
   │
   └── favorite_user_id → User B
```

This means User A can favorite User B without requiring User B to favorite User A.

---

# 🚫 Blocking

Users can block other users.

The blocking relationship is stored in:

```text
user_blocks
```

Blocking can affect:

* User discovery
* Chat interaction
* User visibility
* Communication

Chat rooms also support blocking:

```text
room_status = blocked
blocked_by
```

---

# 🚩 Reporting

Users can report another user for moderation.

Reports are stored in the `user_reports` table.

```text
user_reports
├── id
├── reporter_id
├── reported_user_id
├── reason
├── description
├── status
├── reviewed_by
├── reviewed_at
├── admin_notes
├── created_at
└── updated_at
```

### Report Statuses

```text
pending
reviewed
resolved
dismissed
```

This allows administrators to review, resolve, or dismiss reports.

---

# 🔔 Device Tokens

Mobile devices register their Firebase Cloud Messaging device token with the Laravel API.

### Register Device Token

```http
POST /api/v1/device-token
```

Example request:

```json
{
    "device_token": "FCM_DEVICE_TOKEN",
    "platform": "android"
}
```

Device tokens are stored in:

```text
device_tokens
├── id
├── user_id
├── device_token
├── platform
├── created_at
└── updated_at
```

A unique device token prevents duplicate device registrations.

Supported platforms currently include:

```text
android
ios
web
```

---

# 🔑 Authentication

Zynvo uses separate authentication mechanisms for different application components.

## Google / Firebase Authentication

The mobile application uses Google authentication through Firebase.

```text
Flutter App
     │
     ▼
Google Sign-In
     │
     ▼
Firebase Authentication
     │
     ▼
Firebase ID Token
     │
     ▼
Laravel API
     │
     ▼
Authenticated User
```

The Laravel API verifies the authenticated user before protected endpoints are processed.

The authenticated user is available to the controller through:

```php
$request->attributes->get('authUser');
```

---

# 🖥️ Admin Panel

Zynvo includes a Laravel-based admin panel for platform management and moderation.

### Admin Capabilities

* Admin authentication
* User management
* User profile management
* Blocked user management
* User reports
* Moderation
* Account status management
* Platform management

The admin panel uses Laravel's session-based web authentication.

The mobile API authentication flow is handled separately.

---

# 🏗️ System Architecture

```text
                         ┌──────────────────────┐
                         │     Flutter App      │
                         │                      │
                         │  Google Login        │
                         │  Profile             │
                         │  Discovery           │
                         │  Favorites           │
                         │  Blocks              │
                         │  Reports             │
                         │  Chat                │
                         └──────────┬───────────┘
                                    │
                     ┌──────────────┴──────────────┐
                     │                             │
                     ▼                             ▼
              ┌──────────────┐              ┌──────────────┐
              │ Laravel API  │              │   Firebase   │
              │              │              │              │
              │ Authentication│             │ Text Chat    │
              │ Profiles     │              │ FCM          │
              │ Discovery    │              │              │
              │ Chat Rooms   │              └──────────────┘
              │ Media        │
              │ Favorites    │
              │ Blocks       │
              │ Reports      │
              └──────┬───────┘
                     │
              ┌──────┴─────────┐
              │                │
              ▼                ▼
        ┌────────────┐   ┌───────────────┐
        │   MySQL    │   │ Private Media │
        │  Database  │   │    Storage    │
        └────────────┘   └───────────────┘


                 ┌──────────────────────┐
                 │    Laravel Admin     │
                 │                      │
                 │ User Management      │
                 │ Reports              │
                 │ Moderation           │
                 └──────────────────────┘
```

---

# 🌐 Application Domains

| Component | Domain                      |
| --------- | --------------------------- |
| Frontend  | `zynvo.syslotech.com`       |
| API       | `zynvo-api.syslotech.com`   |
| Admin     | `zynvo-admin.syslotech.com` |

---

# 🛠️ Technology Stack

## Backend

* Laravel
* PHP
* MySQL
* REST API
* Firebase Authentication
* Laravel Authentication
* JWT/API authentication where applicable

## Mobile

* Flutter
* Google Authentication
* Firebase
* REST API
* Image upload
* Voice message upload

## Services

* Firebase
* Firebase Cloud Messaging
* Private media storage
* MySQL

---

# 🗄️ Database Architecture

The primary application database is MySQL.

## Core Tables

```text
users
user_profiles
user_photos
device_tokens

chat_rooms
messages

user_favorites
user_blocks
user_reports

notifications
notification_logs
settings
```

## User Architecture

```text
users
  │
  ├── user_profiles
  │
  ├── user_photos
  │
  ├── device_tokens
  │
  ├── user_favorites
  │
  ├── user_blocks
  │
  └── user_reports
```

## Chat Architecture

```text
users
   │
   ▼
chat_rooms
   │
   ▼
messages
```

---

# 👤 User Data Architecture

Zynvo separates authentication/account information from public profile information.

```text
┌──────────────────────────────┐
│            users             │
├──────────────────────────────┤
│ id                           │
│ uid                          │
│ name                         │
│ email                        │
│ password                     │
│ phone_number                 │
│ photo                         │
│ gender                       │
│ status                       │
│ user_type                    │
│ is_blocked                   │
│ is_deleted                   │
│ profile_status               │
│ profile_status_at            │
│ last_login_at                │
│ deleted_at                   │
│ deleted_by                   │
│ created_at                   │
│ updated_at                   │
└──────────────┬───────────────┘
               │
               │ 1:1
               ▼
┌──────────────────────────────┐
│        user_profiles         │
├──────────────────────────────┤
│ id                           │
│ user_id                      │
│ nickname                     │
│ gender                       │
│ age                          │
│ looking_for                  │
│ age_range_min                │
│ age_range_max                │
│ online_status                │
│ status                       │
│ created_at                   │
│ updated_at                   │
└──────────────┬───────────────┘
               │
               │ 1:N
               ▼
┌──────────────────────────────┐
│         user_photos          │
├──────────────────────────────┤
│ id                           │
│ user_id                      │
│ photo_url                    │
│ position                     │
│ is_primary                   │
│ created_at                   │
│ updated_at                   │
└──────────────────────────────┘
```

---

# 🖼️ Private Media Storage

Chat media and private user information should not be exposed through unrestricted public URLs.

Private media is stored outside the public web-accessible directory.

Example:

```text
storage/app/
│
└── chat-media/
    │
    ├── 1/
    │   ├── unique-file.jpg
    │   └── unique-file.m4a
    │
    └── 2/
        └── unique-file.jpg
```

Uploaded filenames are generated using unique random values to avoid filename collisions.

For example:

```php
Str::random(20)
```

The database stores the media path rather than storing the actual binary file.

---

# 📡 API Structure

The API is versioned using:

```text
/api/v1
```

## Main API Modules

```text
Authentication
User
Profile
Device Token
Discovery
Chat Rooms
Chat Media
Favorites
Blocks
Reports
```

---

# 📡 Chat APIs

### Chat Rooms

```http
GET  /api/v1/chat-rooms
POST /api/v1/chat-rooms
```

### Chat Messages

```http
GET /api/v1/chat-rooms/{chatRoom}/messages
```

Text messages are handled directly through Firebase in the current architecture.

### Chat Media

```http
POST /api/v1/chat-rooms/{chatRoom}/media
```

The media API handles:

* Image uploads
* Voice uploads
* Media validation
* Private storage
* Message metadata
* Chat room last-message updates

---

# 🔍 Discovery & Chat Relationship

The discovery system is designed to avoid repeatedly showing users who already have an established chat relationship.

Example:

```text
User A
   │
   ├── Discover User B
   │
   ├── Create Chat Room
   │
   └── Chat
          │
          ▼
     User B can be
     excluded from
     future discovery
```

Blocked users can also be excluded from discovery.

Favorites are maintained independently from chat relationships.

---

# 🛡️ Safety & Moderation

Zynvo includes several safety mechanisms:

* User blocking
* User reporting
* Anonymous profiles
* Private account information
* Age restrictions
* Account status management
* Chat room blocking
* Soft deletion
* Admin moderation
* Report review workflow

---

# 📱 Screenshots

Selected screenshots are included in the case-study repository.

## Login

![Zynvo Login](screenshots/login.png)

## Discovery

![Zynvo Discovery](screenshots/discovery.png)

## Chat

![Zynvo Chat](screenshots/chat.png)

## Chat List

![Zynvo Chat List](screenshots/chat-list.png)

## Profile

![Zynvo Profile](screenshots/profile.png)

## Admin Panel

![Zynvo Admin Panel](screenshots/admin-dashboard.png)

> Screenshots may be updated as the application UI evolves.

---

# 🗺️ Development Roadmap

## Phase 1 — Foundation

* [x] Laravel backend
* [x] MySQL database
* [x] Domain structure
* [x] API structure
* [x] User authentication
* [x] Admin authentication
* [x] Admin panel foundation

## Phase 2 — Profile

* [x] User profile
* [x] Anonymous nickname
* [x] Age
* [x] Gender
* [x] Profile preferences
* [x] Application photos
* [x] Private real/Google profile information

## Phase 3 — Discovery

* [x] User discovery
* [x] Age filtering
* [x] Gender filtering
* [x] Profile-based preferences
* [x] Blocked-user filtering
* [x] Existing chat filtering
* [x] Anonymous profile response

## Phase 4 — Chat

* [x] Chat room creation
* [x] Chat room listing
* [x] Firebase text messaging
* [x] Image upload
* [x] Voice upload
* [x] Chat media storage
* [x] Chat history architecture
* [x] Room status management

## Phase 5 — Social Features

* [x] Favorites
* [x] Block users
* [x] Report users

## Phase 6 — Safety & Moderation

* [x] User reports
* [x] Report statuses
* [x] Admin moderation structure
* [x] User restrictions
* [x] Account status management

## Phase 7 — Production

* [ ] Push notification workflow
* [ ] Private media access endpoints
* [ ] Media optimization
* [ ] Performance optimization
* [ ] Monitoring
* [ ] Analytics
* [ ] Production hardening
* [ ] Google Play release

---

# 📂 Case Study Structure

```text
zynvo-case-study/
│
├── README.md
│
├── docs/
│   ├── admin-panel.md
│   ├── api.md
│   ├── architecture.md
│   ├── database.md
│   ├── privacy.md
│   └── roadmap.md
│
├── screenshots/
│   ├── login.png
│   ├── discovery.png
│   ├── chat.png
│   ├── chat-list.png
│   ├── profile.png
│   └── admin-dashboard.png
│
└── LICENSE
```

---

# 🎯 Project Goals

Zynvo is being developed with the following goals:

* Provide simple anonymous communication
* Make discovering new people easy
* Keep real user information private
* Give users control over their public profile
* Support text, voice, and image communication
* Build safety and moderation into the platform
* Create a scalable backend architecture
* Separate private and public user data
* Provide a clean mobile experience

---

# 🔐 Security Considerations

Security is considered throughout the application architecture.

### Authentication & Authorization

* Firebase ID token verification
* Authentication middleware
* Laravel authentication
* Admin authorization
* Protected API endpoints

### Data Security

* Password hashing
* Request validation
* Input validation
* Private user information
* Private media storage
* Unique media filenames
* Soft deletion

### Platform Safety

* User blocking
* User reporting
* Age restriction
* Account status management
* Admin moderation

Production secrets are intentionally excluded from the public repository.

The following must never be committed:

```text
.env
Firebase private keys
API secrets
JWT secrets
Database credentials
Service account credentials
Production passwords
```

---

# 🚧 Current Development Status

Zynvo is currently in active development.

### Implemented

```text
✅ Google/Firebase Authentication
✅ User Management
✅ Anonymous Profiles
✅ User Profiles
✅ User Photos
✅ Device Tokens
✅ User Discovery
✅ Chat Room Creation
✅ Chat Room Listing
✅ Firebase Text Chat
✅ Image Upload
✅ Voice Upload
✅ Private Chat Media Storage
✅ Favorites
✅ Blocking
✅ User Reporting
✅ Admin Foundation
```

### In Progress

```text
🔄 Push Notifications
🔄 Private Media Access
🔄 Production Optimization
🔄 Monitoring
🔄 Analytics
🔄 Mobile UI Refinement
```

---

# 👨‍💻 Developer

## Mukesh Prajapat

Full Stack Developer specializing in:

* Laravel
* PHP
* Shopify
* WordPress
* REST APIs
* JavaScript
* React
* Mobile application backends
* API integrations
* Payment integrations

---

# 📄 License

This repository contains project documentation and case-study materials for Zynvo.

The actual Zynvo application source code is maintained separately in a private repository.

---

# ⭐ About Zynvo

**Zynvo — Meet. Chat. Connect. Anonymously.**

A privacy-focused social chat platform built around anonymous discovery, meaningful conversations, private user information, and user-controlled profile visibility.

```

**One important recommendation before you commit:** don't mark a feature `[x]` just because its database/controller exists. For the case study, `[x]` should mean you've actually tested the complete flow from the Flutter app through the API/Firebase and back. This will keep the README accurate as the project progresses.
```
