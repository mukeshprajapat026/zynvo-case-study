# 💜 Zynvo
#### Anonymous Social Chat Platform

### Anonymous Social Chat Platform

**Meet. Chat. Connect. Anonymously.**

Zynvo is a privacy-focused social chat platform designed to help users discover and connect with new people while maintaining control over the information they share.

Users can discover compatible people based on age and gender preferences, skip users they are not interested in, create chat rooms, exchange messages through Firebase, share images and voice messages, and use built-in favorite, block, and report functionality.

> 🚧 **Status:** Active Development

---

## ✨ Core Features

### 👤 Anonymous Profiles

Zynvo separates private account information from the public profile shown inside the application.

Private authentication information such as the user's real Google account name, email, and authentication information is stored separately from the public profile.

The public profile uses:

- Anonymous/display nickname
- Age
- Gender
- Profile photos
- Matching preferences
- Online status

Users do not need to expose their real Google identity to other users.

---

## 🔐 Privacy-Focused Architecture

Zynvo follows a separation between **private account data** and **public profile data**.

```text
                    User Authentication
                           │
                           ▼
                     ┌───────────┐
                     │   users   │
                     │           │
                     │ Real name │
                     │ Email     │
                     │ Firebase  │
                     │ identity  │
                     └─────┬─────┘
                           │
                           │ 1 : 1
                           ▼
                  ┌─────────────────┐
                  │  user_profiles  │
                  │                 │
                  │ Nickname        │
                  │ Age             │
                  │ Gender          │
                  │ Preferences     │
                  │ Online status   │
                  └────────┬────────┘
                           │
                           │ 1 : many
                           ▼
                    ┌────────────┐
                    │ user_photos│
                    │            │
                    │ Public     │
                    │ profile    │
                    │ photos     │
                    └────────────┘
````

The real Google profile information is not used as the public Zynvo profile identity.

---

# 🎯 User Discovery

Zynvo provides a discovery system based on the user's profile preferences.

The discovery API automatically uses the authenticated user's profile settings rather than requiring the mobile application to send age and gender filters on every request.

### Discovery considers:

* Age range
* Gender preference
* Active users
* Block relationships
* Skipped users
* Existing chat relationships
* Deleted accounts
* Account status

### Discovery flow

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
     ├───────────────┐
     ▼               ▼
   Chat             Skip
     │               │
     ▼               ▼
 Firebase          Store Skip
 Chat                │
                     ▼
              Exclude from
               future discovery
```

---

# ⏭️ Skip Users

Users can skip another user from the discovery screen.

A dedicated `user_skips` table stores the relationship:

```text
user_id
skipped_user_id
created_at
updated_at
```

Example:

```text
User 32 → skips → User 9
```

After that, User 9 is excluded from User 32's discovery results.

The database also prevents duplicate skip records.

---

# 💬 Chat Architecture

Zynvo uses a hybrid chat architecture.

### Text messaging

Text messages are handled directly between the Flutter application and Firebase.

```text
Flutter App
     │
     ▼
  Firebase
     │
     ▼
Text Messages
```

Laravel does not provide a REST API for sending normal text messages.

### Chat rooms

Laravel manages chat room relationships:

```text
User
  │
  ├── Chat Room
  │      │
  │      ├── User 1
  │      └── User 2
  │
  └── Chat history
```

### Chat media

Images and voice messages are uploaded through Laravel.

```text
Flutter App
     │
     │ multipart/form-data
     ▼
Laravel API
     │
     ▼
Private Storage
     │
     ▼
messages table
```

The database stores the media path and message metadata.

---

# 🖼️ Chat Media

Supported chat media:

* 🖼️ Images
* 🎤 Voice messages

The media upload API is:

```http
POST /api/v1/chat-rooms/{chatRoom}/media
```

The uploaded media is stored using a generated filename instead of the original filename.

Example:

```text
chat-media/
└── 12/
    └── xK8s92LmPq4Tn7Yw.jpg
```

The media path is stored with the corresponding chat message.

---

# ❤️ Favorites

Users can add another user to their favorites.

### APIs

```http
GET    /api/v1/users/favorites
POST   /api/v1/users/{user}/favorite
DELETE /api/v1/users/{user}/favorite
```

Favorite relationships are stored separately from the main user account.

---

# 🚫 Block System

Users can block other users.

### APIs

```http
GET    /api/v1/users/blocked
POST   /api/v1/users/{user}/block
DELETE /api/v1/users/{user}/block
```

Blocked users are excluded from discovery and cannot interact normally with the blocking user.

---

# 🚩 Report System

Users can report another user from the application.

```http
POST /api/v1/users/{user}/report
```

Reports are stored for moderation and administrative review.

---

# 👤 Profile Management

Users can manage their public Zynvo profile.

### Profile APIs

```http
GET /api/v1/user/profile
PUT /api/v1/user/profile
```

### Profile photo APIs

```http
POST   /api/v1/user/photos
DELETE /api/v1/user/photos/{photo}
PUT    /api/v1/user/photos/{photo}/primary
```

Users can upload multiple profile photos and choose which photo is their primary profile photo.

---

# 🗑️ Account Deletion

Zynvo uses soft deletion for user accounts.

```http
DELETE /api/v1/user/account
```

Deleted accounts remain in the database through Laravel's soft-delete mechanism while no longer being treated as active users.

The system also supports registering again with the same email after an account has been deleted, creating a new active user account.

---

# 🔔 Device Tokens

The mobile application registers its Firebase Cloud Messaging device token with Laravel.

```http
POST /api/v1/user/device-token
```

Example request:

```json
{
    "device_token": "FCM_DEVICE_TOKEN",
    "platform": "android"
}
```

Device tokens are stored separately from user account information.

---

# 🔑 Authentication

Zynvo uses Firebase Authentication for mobile users.

```text
Flutter
   │
   ▼
Firebase Authentication
   │
   ▼
Firebase ID Token
   │
   ▼
Laravel Firebase Middleware
   │
   ▼
Authenticated User
```

The Laravel API extracts the authenticated Firebase user and synchronizes the account with the application's `users` table.

---

# 🏗️ Architecture

```text
                    ┌──────────────────────┐
                    │     Flutter App      │
                    │                      │
                    │  Authentication      │
                    │  Profile             │
                    │  Discovery           │
                    │  Favorites           │
                    │  Block               │
                    │  Report              │
                    │  Chat UI             │
                    └──────────┬───────────┘
                               │
                ┌──────────────┴──────────────┐
                │                             │
                ▼                             ▼
       ┌────────────────┐            ┌────────────────┐
       │ Firebase       │            │ Laravel REST   │
       │                │            │ API            │
       │ Authentication │            │                │
       │ Text Chat      │            │ User Sync      │
       │ FCM            │            │ Profile        │
       └────────────────┘            │ Discovery      │
                                     │ Favorites      │
                                     │ Block/Report   │
                                     │ Chat Rooms     │
                                     │ Media Upload   │
                                     └───────┬────────┘
                                             │
                         ┌───────────────────┼───────────────────┐
                         ▼                   ▼                   ▼
                  ┌────────────┐      ┌────────────┐     ┌────────────┐
                  │   MySQL    │      │  Private   │     │  Firebase  │
                  │            │      │  Storage   │     │ Services   │
                  │ Users      │      │            │     │            │
                  │ Profiles   │      │ Chat Media │     │ Auth       │
                  │ Photos     │      │            │     │ FCM        │
                  │ Chats      │      └────────────┘     │ Chat       │
                  │ Favorites  │                         └────────────┘
                  │ Blocks     │
                  │ Reports    │
                  │ Skips      │
                  └────────────┘
```

---

# 🗄️ Database Architecture

The primary database is MySQL.

### Core tables

```text
users
│
├── user_profiles
├── user_photos
├── device_tokens
├── user_skips
├── user_favorites
├── user_blocks
├── user_reports
│
└── chat_rooms
      │
      └── messages
```

### Main tables

| Table            | Purpose                                        |
| ---------------- | ---------------------------------------------- |
| `users`          | Private account and authentication information |
| `user_profiles`  | Public Zynvo profile                           |
| `user_photos`    | Public profile photos                          |
| `device_tokens`  | FCM device tokens                              |
| `user_skips`     | Users skipped during discovery                 |
| `user_favorites` | Favorite relationships                         |
| `user_blocks`    | Block relationships                            |
| `user_reports`   | User reports                                   |
| `chat_rooms`     | User-to-user chat rooms                        |
| `messages`       | Chat and media message metadata                |

---

# 🌐 API Structure

All mobile APIs use the `/api/v1` prefix.

## User

```http
POST   /api/v1/user/sync
POST   /api/v1/user/device-token
DELETE /api/v1/user/account
```

## Profile

```http
GET    /api/v1/user/profile
PUT    /api/v1/user/profile
POST   /api/v1/user/photos
DELETE /api/v1/user/photos/{photo}
PUT    /api/v1/user/photos/{photo}/primary
```

## Discovery

```http
GET  /api/v1/discover
GET  /api/v1/users/{user}
POST /api/v1/users/{user}/skip
```

## Chat

```http
GET  /api/v1/chat-rooms
POST /api/v1/chat-rooms

GET  /api/v1/chat-rooms/{chatRoom}/messages

POST /api/v1/chat-rooms/{chatRoom}/media
```

## Favorites

```http
GET    /api/v1/users/favorites
POST   /api/v1/users/{user}/favorite
DELETE /api/v1/users/{user}/favorite
```

## Block

```http
GET    /api/v1/users/blocked
POST   /api/v1/users/{user}/block
DELETE /api/v1/users/{user}/block
```

## Reports

```http
POST /api/v1/users/{user}/report
```

---

# 🛡️ Safety & Moderation

Zynvo includes several features designed to create a safer social environment.

* Anonymous public profiles
* Age restrictions
* User blocking
* User reporting
* Discovery filtering
* Skipped-user filtering
* Existing-chat filtering
* Account status management
* Soft deletion
* Admin moderation
* Private media storage

---

# 🖥️ Admin Panel

Zynvo includes a Laravel-based administration panel.

### Admin functionality

* 🔐 Admin authentication
* 👥 User management
* 🚫 Blocked user management
* 🚩 User report management
* 👤 Profile management
* 🛡️ Moderation
* 📊 Platform management

The admin interface uses Laravel's browser/session authentication while the mobile application uses Firebase authentication.

---

# 🛠️ Technology Stack

## Backend

* Laravel
* PHP
* MySQL
* REST APIs
* Laravel Eloquent
* Laravel Soft Deletes

## Mobile

* Flutter
* Android
* Firebase Authentication
* Firebase Chat
* Firebase Cloud Messaging

## Storage

* Laravel private storage
* Chat media storage
* Profile photo storage

## Authentication

* Firebase Authentication
* Firebase ID Tokens
* Laravel authentication middleware
* Laravel session authentication for admin

---

# 📱 Application Screens

Selected application screenshots are included in this case study.

### Login

![Zynvo Login](screenshots/login.png)

### Discovery

![Zynvo Discovery](screenshots/discovery.png)

### Chat

![Zynvo Chat](screenshots/chat.png)

### Chat List

![Zynvo Chat List](screenshots/chat-list.png)

### Profile

![Zynvo Profile](screenshots/profile.png)

> Screenshot files are added as the case study documentation is finalized.

---

# 🗺️ Development Progress

## Phase 1 — Foundation

* [x] Laravel backend
* [x] MySQL database
* [x] API versioning
* [x] Firebase authentication
* [x] User synchronization
* [x] Admin authentication
* [x] Admin panel foundation

## Phase 2 — User Profile

* [x] Anonymous nickname
* [x] Age
* [x] Gender
* [x] Profile photos
* [x] Primary profile photo
* [x] Matching preferences
* [x] Profile update
* [x] Account deletion

## Phase 3 — Discovery

* [x] User discovery
* [x] Age filtering
* [x] Gender filtering
* [x] Existing-chat exclusion
* [x] Blocked-user exclusion
* [x] Skipped-user exclusion
* [x] User profile details
* [x] Skip functionality

## Phase 4 — Chat

* [x] Chat room creation
* [x] Chat room listing
* [x] Chat history API
* [x] Firebase text messaging
* [x] Image message upload
* [x] Voice message upload
* [x] Private chat media storage

## Phase 5 — Social & Safety

* [x] Favorites
* [x] Block
* [x] Unblock
* [x] Report
* [x] User moderation
* [x] Device token management

## Phase 6 — Production Readiness

* [ ] Performance optimization
* [ ] Media optimization
* [ ] Monitoring
* [ ] Analytics
* [ ] Production hardening
* [ ] Google Play release

---

# 🔒 Security Considerations

Security and privacy are considered throughout the architecture.

* Firebase authentication
* Authentication middleware
* Input validation
* Laravel request validation
* Soft deletion
* Account status checks
* Block checks
* Report functionality
* Private media storage
* Generated media filenames
* Sensitive authentication information kept separate from public profiles
* Production credentials excluded from the repository

The following are intentionally not included in this public case study:

```text
.env
Firebase private keys
Database credentials
JWT secrets
API keys
Production passwords
```

---

# 🚧 Project Status

**Active Development**

Zynvo is currently under active development.

The core backend architecture, user management, anonymous profile system, discovery, social actions, chat rooms, Firebase messaging integration, and media upload functionality have been implemented.

The architecture and UI may continue to evolve as the project moves toward production.

---

# 🎯 Project Goals

Zynvo is designed around the following goals:

* Provide anonymous social discovery
* Protect users' real identities
* Make discovering compatible people simple
* Give users control over their public profile
* Support multiple communication methods
* Provide built-in safety and moderation tools
* Build a scalable Laravel backend
* Integrate Firebase services where real-time communication is required
* Provide a clean mobile experience

---

# 📂 Case Study Structure

```text
zynvo-case-study/
│
├── README.md
│
├── docs/
│   ├── case-study.md
│   ├── architecture.md
│   ├── database.md
│   ├── api.md
│   ├── admin-panel.md
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

# 👨‍💻 Developer

## Mukesh Prajapat

Full Stack Developer specializing in:

* Laravel
* PHP
* REST API Development
* Shopify
* WordPress
* JavaScript
* React
* Mobile Application Backends
* Firebase Integrations
* API Integrations
* Payment Integrations

### Project Focus

For Zynvo, the primary development focus includes:

* Laravel backend architecture
* REST API development
* MySQL database design
* Firebase integration
* Authentication
* Anonymous profile architecture
* Discovery logic
* Chat infrastructure
* Media handling
* User safety features

---

# 📄 License

This repository contains project documentation and case-study materials for Zynvo.

The actual Zynvo application source code is maintained separately in a private repository.

---

# 💜 About Zynvo

**Zynvo — Meet. Chat. Connect. Anonymously.**

A privacy-focused social chat platform built around anonymous discovery, user-controlled profile visibility, real-time conversations, and built-in safety features.

````