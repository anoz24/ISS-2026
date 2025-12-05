# **2️⃣ Backend Architecture (Recommended)**

```
/src
 ├── config/            → env variables, DB config, AES keys, JWT secrets
 ├── middleware/        → rate limiter, authentication, validation, error handling
 ├── modules/
 │     ├── auth/        → login, signup, SSO integration, bcrypt, JWT
 │     ├── todos/       → CRUD operations, encryption for descriptions
 │     └── users/       → user profile, role checks
 ├── utils/             → AES encryption, token helpers, sanitization
 └── app.js             → Express app
```

---

## **How This Architecture Meets All Security Requirements**

### **✔ Password Hashing (bcrypt)**

In `modules/auth/`

* `POST /auth/signup`
* Hash passwords using `bcrypt`
* Store salted hash in DB

---

### **✔ AES Encryption (Sensitive To-Do Description)**

In `utils/encryption.js`

Use Node’s built-in module:

```js
import crypto from 'crypto';
```

Encrypt data **before storing in DB**:

* Encrypt the **todo description**
* Store the encrypted version
* Decrypt when retrieving

This satisfies **Encryption at Rest**.

---

### **✔ JWT Authentication**

In `modules/auth/`

* `jsonwebtoken` for access + refresh tokens
* Refresh token rotation stored in DB
* Middleware verifies the JWT + expiry

---

### **✔ SSO Login (OIDC)**

Using **Passport.js** (OIDC strategy)
or Auth0 SDK for Node.

Flow:

```
Frontend → SSO Login Button
        → Redirect to Provider (Google/Auth0)
        → Provider returns authorization code
        → Backend exchanges for ID token
        → Create/Find user in DB
        → Issue JWT access token
```

---

### **✔ SQL Injection Prevention**

Use **parameterized queries** with `pg` library OR Prisma.

Example:

```js
const result = await db.query(
  "INSERT INTO todos (user_id, title, desc) VALUES ($1, $2, $3)",
  [userId, title, encryptedDesc]
);
```

No string concatenation → **safe**.

---

### **✔ Prevent DoS**

Middleware in `/middleware/security.js`:

* `express-rate-limit`
* Limit login attempts
* Limit note creation
* Payload size limit
* Timeout handlers

---

### **✔ Prevent XSS**

Handled in 2 places:

### **Backend**

* Use `helmet` to set CSP header
* Sanitize user text input
* Escape output

### **Frontend**

* Never use `dangerouslySetInnerHTML`
* Encode any dynamic HTML

---

# **3️⃣ API Endpoints (Simple & Clean)**

### **Auth**

```
POST /auth/signup
POST /auth/login
GET  /auth/sso/google
GET  /auth/sso/google/callback
POST /auth/refresh
POST /auth/logout
```

### **Todos**

```
GET    /todos
POST   /todos
PATCH  /todos/:id
DELETE /todos/:id
```

Each to-do contains:

* `title`
* `description` (AES-encrypted)
* `created_at`
* `completed`
* `user_id`

---

# **4️⃣ Database Schema (Simple & Secure)**

```
users
---------
id (PK)
email (unique)
password_hash
created_at
sso_provider
sso_id

todos
---------
id (PK)
user_id (FK)
title
description_encrypted
completed (boolean)
created_at

refresh_tokens
---------
id (PK)
user_id (FK)
token_hash     (hashed refresh token)
expires_at
revoked (bool)
```

---
