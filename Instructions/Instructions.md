
---

# ✅ **Task Instructions for Each Team Member (To-Do App Project)**

## **1. Encryption Specialist (AES Encryption for Sensitive Data)**

**Goal:** Implement AES encryption/decryption for sensitive fields (e.g., private notes).

### **Instructions**

* Create a module `encryption.js`.
* Use Node’s `crypto` library or `crypto-js`.
* Implement:

  * `encrypt(text)` → returns encrypted string
  * `decrypt(ciphertext)` → returns plaintext
* Store the AES key in an **environment variable** (`AES_SECRET`).
* Integrate encryption only on fields marked “sensitive” in the Todo model.
* Ensure decrypted data is never stored in logs.
* Write small tests to confirm:

  * Encryption works
  * Decryption retrieves original data

---

## **2. Authentication Leader (JWT Tokens + Login/Signup Flow)**

**Goal:** Implement signup, login, hashed passwords, and JWT token authentication.

### **Instructions**

* Create routes:

  * `POST /auth/signup`
  * `POST /auth/login`
* Use **bcrypt** to hash user passwords before storing.
* Use **JWT** for creating access tokens.
* Store JWT secret in `.env` → `JWT_SECRET`.
* Create middleware `authMiddleware.js`:

  * Read `Authorization: Bearer <token>`
  * Verify JWT
  * Attach `req.user = decodedPayload`
* Protect routes:

  * `/todos/*`
* Do **not** include password in any response.

---

## **3. OAuth/SSO Engineer (Google Login Integration)**

**Goal:** Enable login via Google as an SSO provider.

### **Instructions**

* Use **Passport.js** with `passport-google-oauth20`.
* Register a Google OAuth app → get `GOOGLE_CLIENT_ID` & `GOOGLE_CLIENT_SECRET`.
* Setup callback route:

  * `/auth/google`
  * `/auth/google/callback`
* On successful login:

  * Check if user exists; if not, create one.
  * Generate a JWT and return it to frontend.
* Do not store Google access tokens.
* Ensure redirect URLs match Google console configuration.

---

## **4. Secure Coding & Backend Hardening Specialist**

**Goal:** Implement protections against SQL Injection, XSS, and DoS.

### **Instructions**

* Use **MongoDB + Mongoose** (already protects from SQL injection).
* Globally add **helmet** middleware for security headers.
* Add **express-rate-limit** for DoS prevention on:

  * Auth routes (high-risk)
  * Todo CRUD routes
* Validate all request bodies with:

  * **zod** or **express-validator**
* Strip HTML from user inputs using `sanitize-html` to prevent stored XSS.
* Ensure errors never leak stack traces in production.

---

## **5. Backend API Developer (Todo CRUD & User Features)**

**Goal:** Build all core To-Do operations (CRUD) using Express + MongoDB.

### **Instructions**

* Create routes:

  * `GET /todos`
  * `POST /todos`
  * `PUT /todos/:id`
  * `DELETE /todos/:id`
* Each todo stores:

  * title
  * description (sensitive → use AES encryption module)
  * status (pending / done)
  * createdAt
  * userId
* Only allow users to access their own todos.
* Use the `authMiddleware` (task #2) to secure routes.
* Never return encrypted text — decrypt before responding.

---

## **6. Frontend Developer (React + Auth + API Integration)**

**Goal:** Build a simple UI for the To-Do app.

### **Instructions**

* Use **React + Vite**.
* Pages required:

  * Login
  * Signup
  * Todo List Page
* Store JWT token in **localStorage**.
* On app load, check if user is logged in → redirect accordingly.
* Implement Todo actions:

  * Add
  * Edit
  * Delete
* Use `fetch` or Axios to call backend routes.
* Sanitize all user inputs before sending to backend.
* Show error messages from backend cleanly.

---


