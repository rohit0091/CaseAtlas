# CaseAtlas
A global, community-driven platform that tracks ongoing cases (criminal, civil, cyber, corporate, regulatory, or social) and provides verified, structured updates to subscribers—aggregated from multiple sources and enriched with community intelligence



# 📁 CaseSphere Backend  
### A Global Community-Driven Case Tracking & Verification Platform

CaseSphere is a secure, role-based backend API designed to track ongoing cases (criminal, civil, cyber, corporate, regulatory, or social) with structured updates and moderated verification.

---

## 🚀 Step 1 — Tech Stack

- Node.js
- Express.js
- MongoDB
- Mongoose
- JWT Authentication
- bcrypt (Password Hashing)
- Role-Based Access Control (RBAC)

---

# 🏗️ Backend Architecture

```
Client
   ↓
Routes
   ↓
Middleware (Auth / Roles)
   ↓
Controllers
   ↓
MongoDB Database
```

---

# 📂 Step 2 — Project Structure

```
casesphere-backend/
│
├── src/
│   ├── app.js
│   ├── server.js
│
│   ├── config/
│   │   └── db.js
│
│   ├── models/
│   │   ├── User.js
│   │   ├── Case.js
│   │   └── CaseUpdate.js
│
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── case.routes.js
│   │   └── update.routes.js
│
│   ├── controllers/
│   │   ├── auth.controller.js
│   │   ├── case.controller.js
│   │   └── update.controller.js
│
│   ├── middleware/
│   │   ├── auth.middleware.js
│   │   └── role.middleware.js
│
├── .env
├── package.json
└── README.md
```

---

# ⚙️ Step 3 — Setup Instructions

## 1️⃣ Install Dependencies

```bash
npm install
```

## 2️⃣ Create `.env` File

```env
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/casesphere
JWT_SECRET=your_secret_key
```

## 3️⃣ Start MongoDB

Make sure MongoDB service is running on:

```
mongodb://127.0.0.1:27017
```

## 4️⃣ Run Development Server

```bash
npm run dev
```

Server runs on:

```
http://localhost:5000
```

---

# 🔐 Step 4 — Authentication System

### Features Implemented

- User Registration
- Secure Password Hashing (bcrypt)
- JWT Token Generation
- Protected Routes
- Role-Based Authorization

### Available Roles

| Role        | Permissions |
|-------------|------------|
| user        | Submit updates, view cases |
| moderator   | Create cases, verify updates |
| admin       | Full system control |

---

# 📁 Step 5 — Case Management

## ➤ Create Case (Moderator/Admin Only)

```
POST /api/cases
```

Requires:

```
Authorization: Bearer <token>
```

---

## ➤ Get All Cases (Public)

```
GET /api/cases
```

---

## ➤ Get Case by ID

```
GET /api/cases/:id
```

---

# 🧾 Step 6 — Timeline & Verification System

This is the core feature of CaseSphere.

---

## ➤ Submit Case Update (Logged-in Users)

```
POST /api/updates/:caseId
```

- Default status: `pending`
- Requires authentication

---

## ➤ Verify / Reject Update (Moderator/Admin)

```
PATCH /api/updates/verify/:updateId
```

Example Body:

```json
{
  "status": "verified"
}
```

---

## ➤ Get Public Case Timeline

```
GET /api/updates/:caseId
```

Returns:
- Only verified updates
- Sorted by latest first

---

# 🧪 Step 7 Testing Guide — Timeline & Verification Workflow

This section demonstrates the complete moderated workflow of the CaseSphere backend.

We will simulate:

1. Moderator creates a case  
2. Normal user submits an update  
3. Moderator verifies the update  
4. Public timeline shows only verified updates  

---

# 🧩 Initial Setup Assumption

Current users in database:

- **Rohit** → role: `moderator`
- No normal users yet

We will create a second user for proper workflow testing.

---

# 🔁 Step 1 — Create a Normal User

### Endpoint

```
POST /api/auth/register
```

### URL

```
http://localhost:5000/api/auth/register
```

### Request Body

```json
{
  "name": "Amit",
  "email": "amit@test.com",
  "password": "password123"
}
```

After this:

- Rohit → moderator
- Amit → user (default role)

---

# 🔐 Step 2 — Login as Both Users

We need two JWT tokens.

---

## 🔵 Login as Moderator (Rohit)

### Endpoint

```
POST /api/auth/login
```

### Request Body

```json
{
  "email": "rohit@test.com",
  "password": "password123"
}
```

Copy the returned token.

Store as:

```
MODERATOR_TOKEN
```

---

## 🟢 Login as Normal User (Amit)

### Endpoint

```
POST /api/auth/login
```

### Request Body

```json
{
  "email": "amit@test.com",
  "password": "password123"
}
```

Copy the returned token.

Store as:

```
USER_TOKEN
```

---

# 📁 Step 3 — Create Case (Moderator Only)

Use:

Authorization → Bearer `MODERATOR_TOKEN`

### Endpoint

```
POST /api/cases
```

### URL

```
http://localhost:5000/api/cases
```

### Request Body

```json
{
  "title": "Crypto Scam Investigation",
  "description": "Investigation into multi-state crypto scam.",
  "category": "cyber",
  "tags": ["crypto", "scam"]
}
```

Response will include:

```json
{
  "case": {
    "_id": "CASE_ID"
  }
}
```

Copy the `_id` value.

Store as:

```
CASE_ID
```

---

# 🧾 Step 4 — Submit Update (Normal User)

Use:

Authorization → Bearer `USER_TOKEN`

### Endpoint

```
POST /api/updates/CASE_ID
```

### Example URL

```
http://localhost:5000/api/updates/65abc123...
```

### Request Body

```json
{
  "content": "Police seized 5 laptops connected to fraud.",
  "sourceLinks": ["https://news.com/article"]
}
```

Response will include:

```json
{
  "update": {
    "_id": "UPDATE_ID",
    "verificationStatus": "pending"
  }
}
```

Copy `_id`.

Store as:

```
UPDATE_ID
```

At this stage:

```
verificationStatus = pending
```

---

# 🔎 Step 5 — Check Public Timeline (Before Verification)

### Endpoint

```
GET /api/updates/CASE_ID
```

### URL

```
http://localhost:5000/api/updates/CASE_ID
```

No token required.

Expected response:

```json
[]
```

This is correct because only verified updates are visible publicly.

---

# 🛂 Step 6 — Verify Update (Moderator)

Use:

Authorization → Bearer `MODERATOR_TOKEN`

### Endpoint

```
PATCH /api/updates/verify/UPDATE_ID
```

### Request Body

```json
{
  "status": "verified"
}
```

Expected response:

```json
{
  "message": "Update verified"
}
```

---

# 🔎 Step 7 — Check Public Timeline Again

### Endpoint

```
GET /api/updates/CASE_ID
```

Now the response should contain:

```json
[
  {
    "content": "Police seized 5 laptops connected to fraud.",
    "verificationStatus": "verified"
  }
]
```

This confirms the moderated workflow is functioning correctly.

---

# 🧠 Workflow Summary

| Step | Actor | Action |
|------|--------|--------|
| 1 | Moderator | Creates case |
| 2 | User | Submits update |
| 3 | System | Marks update as pending |
| 4 | Moderator | Verifies update |
| 5 | Public | Sees verified update |

---

# 🗂 Optional: Reset Database (Clean Start)

If needed, open terminal:

```bash
mongosh
```

Then run:

```javascript
use casesphere

db.users.deleteMany({})
db.cases.deleteMany({})
db.caseupdates.deleteMany({})
```

⚠️ This will permanently delete all data.

---

# ✅ Result

After completing these steps, the system demonstrates:

- JWT Authentication
- Role-Based Access Control
- Secure Update Submission
- Moderated Verification System
- Public Verified Timeline Filtering

The backend now supports a complete, structured case-tracking workflow.


# 🗄️ Database Models

## 👤 User Schema

- name
- email
- password (hashed)
- role
- reputationScore
- subscribedCases
- timestamps

---

## 📁 Case Schema

- title
- description
- category
- status
- verified
- createdBy
- tags
- timestamps

---

## 🧾 CaseUpdate Schema

- caseId
- content
- sourceLinks
- submittedBy
- verificationStatus
- verifiedBy
- timestamps

---

# 🔒 Security Features

- JWT-based authentication
- Password hashing with bcrypt
- Role-Based Access Control
- Protected routes using middleware
- Secure token verification

---

# 🧪 API Testing

Tested using Postman:

- Registration & Login
- Token validation
- Role authorization
- Case creation
- Update submission
- Update verification
- Public timeline retrieval

---

# 🎓 Academic Value

This project demonstrates:

- REST API design principles
- Middleware architecture
- Secure authentication systems
- Role-based authorization
- Structured NoSQL data modeling
- Moderated workflow implementation

Suitable for:

- Final Year Project
- Backend Systems Coursework
- Secure API Development Portfolio

---

# 📌 Current Status

- ✅ Step 1 — Project Setup  
- ✅ Step 2 — MongoDB Integration  
- ✅ Step 3 — Schema Design  
- ✅ Step 4 — JWT Authentication  
- ✅ Step 5 — Case Management  
- ✅ Step 6 — Timeline & Verification System  

Backend foundation is fully functional.

---

# 🏁 Conclusion

CaseSphere backend is a scalable, secure, and modular case-tracking API that enables structured community-driven updates with moderation and verification mechanisms.

Built with production-style architecture and academic clarity.
