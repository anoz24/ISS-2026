# ISS-2026
# Secure Web Application — Team Project Documentation

## **1. Team Task Distribution (6 Members)**

### **Team Member 1 — Backend Lead (Authentication & Password Security)**

* Implement user registration & login
* Password hashing using **bcrypt**
* Issue JWT access + refresh tokens
* Implement refresh-token rotation & logout
* Add input validation (schemas)

---

### **Team Member 2 — Encryption & Sensitive Data Handling**

* Implement **AES encryption/decryption** for sensitive fields (e.g., notes, secret info)
* Design DB model for encrypted columns
* Secure key management via environment variables
* Create example endpoint showcasing encrypted DB data

---

### **Team Member 3 — Database & Security Hardening**

* Design relational database schema (PostgreSQL or MySQL)
* Prevent SQL injection using **parameterized queries**
* Apply DB user permissions (least privilege)
* Add pagination & limits to prevent DoS
* Assist with query optimization

---

### **Team Member 4 — SSO & Identity Provider Integration**

* Integrate **SSO using OIDC** (Keycloak/Auth0/Google Identity)
* Implement OAuth login with redirect & callback
* Validate tokens (issuer, audience, signature)
* Map SSO users to local DB profiles
* Add “Login with Google/SSO” UI button

---

### **Team Member 5 — Frontend Security & XSS Prevention**

* Build frontend UI (React or HTML + JS)
* Sanitize/escape user inputs
* Prevent XSS using:

  * Avoiding `innerHTML`
  * Using sanitizers (DOMPurify if needed)
* Apply CSP security headers from backend
* UI for login, SSO login, dashboard, encrypted-note viewer

---

### **Team Member 6 — DevOps / Testing / Secure Deployment**

* Add rate limiting (prevent DoS attacks)
* Apply security middleware (Helmet / FastAPI Security)
* Configure HTTPS locally
* Write tests for:

  * SQL injection attempts
  * XSS payloads
  * Token expiration and refresh
* Create Docker-compose for:

  * Backend
  * Database
  * Keycloak (optional)

---

## **2. Tech Stack Comparison Table**

| Feature / Need               | **Node.js (Express)**                | **Python (FastAPI)**          |
| ---------------------------- | ------------------------------------ | ----------------------------- |
| **Language**                 | JavaScript/TypeScript                | Python                        |
| **API Framework**            | Express.js                           | FastAPI                       |
| **Password Hashing**         | `bcrypt` / `bcryptjs`                | `bcrypt` / `passlib`          |
| **JWT Handling**             | `jsonwebtoken`                       | `PyJWT`, `fastapi-jwt-auth`   |
| **AES Encryption**           | Node `crypto`                        | `cryptography`                |
| **SQL Injection Protection** | `pg` (parameterized queries), Prisma | SQLAlchemy / asyncpg          |
| **SSO Integration**          | Passport.js (OIDC), Auth0 SDK        | python-keycloak / Authlib     |
| **Rate Limiting**            | `express-rate-limit`                 | `slowapi`                     |
| **Security Headers**         | `helmet`                             | Starlette Security Middleware |
| **CORS Handling**            | `cors` package                       | FastAPI CORS middleware       |

---

## **3. Simple System Architecture Overview**

```
                ┌──────────────────────────────┐
                │          Frontend            │
                │ (React / HTML + JS)          │
                │------------------------------│
                │ Login / Signup               │
                │ Login with SSO               │
                │ Create/View Tasks & Notes    │
                └───────────────┬──────────────┘
                                │ HTTPS
                                ▼
                ┌──────────────────────────────┐
                │           Backend            │
                │ (Node.js Express / FastAPI)  │
                │------------------------------│
                │ Auth: bcrypt + JWT           │
                │ SSO: Keycloak/Auth0          │
                │ AES encrypt sensitive data   │
                │ Validate input (anti-DoS)    │
                │ Rate limiting (anti-DoS)     │
                │ Set CSP headers (anti-XSS)   │
                └───────────────┬──────────────┘
                                │ SQL (safe queries)
                                ▼
                  ┌────────────────────────┐
                  │       Database         │
                  │     (PostgreSQL)       │
                  │------------------------│
                  │ users (hashed pw)      │
                  │ refresh_tokens         │
                  │ tasks                  │
                  │ notes (AES encrypted)  │
                  └────────────────────────┘


                ┌────────────────────────┐
                │ Identity Provider (IdP)│
                │  (Keycloak / Auth0)    │
                │------------------------│
                │ Handles SSO login      │
                └────────────────────────┘
```

---

