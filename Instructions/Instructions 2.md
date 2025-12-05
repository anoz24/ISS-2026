
## Project assumptions

* Project root uses Node.js + Express.
* There is an existing auth system that issues JWT access tokens and a middleware `authMiddleware` that sets `req.user = { id, email, ... }`.
* AES encryption utility will be created as `src/utils/encryption.js` (or integrated with existing one) and uses `AES_SECRET` from env.
* Use `pg` (node-postgres) for DB access and parameterized queries.
* Application uses environment variables loaded via `dotenv`.

---

## Files to create / update

Create or update these files (paths relative to project root):

* `src/modules/todos/controller.js` — request handlers
* `src/modules/todos/routes.js` — Express router for `/todos`
* `src/modules/todos/service.js` — DB access + business logic
* `src/modules/todos/schema.js` — validation schemas (Zod or Joi)
* `src/utils/encryption.js` — `encrypt(text)` / `decrypt(ciphertext)`
* `src/middleware/rateLimit.js` — rate-limiter middleware for todo endpoints (if not existing)
* `src/db/index.js` — exported `query(text, params)` wrapper using `pg` (if not existing)
* `migrations/001_create_todos.sql` — DB migration
* `tests/todos.test.js` — tests (use Jest or preferred test runner)
* Add router mounting in `src/app.js` (or wherever routes are registered):

  ```js
  import todosRouter from './modules/todos/routes';
  app.use('/todos', authMiddleware, todosRouter);
  ```

---

## Environment variables required

Add these to `.env` (agent should read from process.env):

```
DATABASE_URL=postgres://user:pass@host:5432/dbname
JWT_SECRET=your_jwt_secret
AES_SECRET=32_byte_base64_or_hex_key
PORT=4000
NODE_ENV=development
```

* `AES_SECRET` must be a 32-byte key (AES-256). For demo it may be base64/hex stored in env; **do not hardcode**.

---

## Database schema / migration SQL

Create table `todos` and `users` assumed to exist.

`migrations/001_create_todos.sql`:

```sql
CREATE TABLE IF NOT EXISTS todos (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  description_encrypted TEXT NOT NULL, -- AES-GCM ciphertext + nonce (encoded)
  completed BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_todos_userid ON todos(user_id);
```

**Note:** Store the encrypted payload as a single base64 string that includes the nonce and auth tag (see encryption util instructions below).

---

## Encryption: `src/utils/encryption.js`

Implement AES-256-GCM authenticated encryption.

API:

```js
// returns base64 string that encodes nonce + ciphertext (and tag if separate)
export function encrypt(plaintext: string): string

// returns decrypted plaintext
export function decrypt(payloadBase64: string): string
```

Implementation notes:

* Use Node `crypto` with `crypto.createCipheriv('aes-256-gcm', key, iv)` where `iv` = 12 random bytes.
* Append the auth tag to the ciphertext (or store separately) and return a single base64 string: `base64(iv + ciphertext + tag)`.
* On decrypt, split iv/ciphertext/tag, set authTag, and use `decipher.final()`.

Security: never log plaintext or keys.

---

## Validation: `src/modules/todos/schema.js`

Use **Zod** or **Joi**. Rules:

* `POST /todos` body:

  * `title`: string, 1–255 chars, required
  * `description`: string, 0–2000 chars, required (sensitive)
* `PATCH /todos/:id` body:

  * `title`: optional string, 1–255 chars
  * `description`: optional string, 0–2000 chars
  * `completed`: optional boolean
* Query params for `GET /todos`:

  * `limit` (optional, int, default 20, max 100)
  * `offset` (optional, int, default 0)

Reject requests with invalid schema — return 400.

Sanitization:

* Trim strings.
* Remove control characters.
* Do **not** do HTML escaping on backend (frontend should avoid injecting HTML). Optionally reject HTML tags if you want to be strict.

---

## Service layer: `src/modules/todos/service.js`

Expose functions:

* `createTodo(userId, { title, description })`

  * Encrypt `description` with encryption util, store in DB.
  * Return created todo (id, title, completed, created_at).
* `getTodosForUser(userId, { limit, offset })`

  * Parameterized SELECT, ORDER BY created_at DESC.
  * Decrypt description before returning to controller.
* `getTodoById(userId, todoId)`

  * Ensure `user_id = userId` filter.
* `updateTodo(userId, todoId, updates)`

  * Only allow updates for owner.
  * If description present, encrypt before writing.
  * Update `updated_at` timestamp.
* `deleteTodo(userId, todoId)`

  * Soft delete not required — hard delete is OK.

Implementation requirements:

* Use parameterized queries (`$1, $2, ...`) via `pg`.
* Catch DB errors and translate to HTTP-friendly errors (e.g., 404 if not found).

Return values:

* Always return decrypted `description` to controllers (never return raw ciphertext).

---

## Controller & routes: `src/modules/todos/controller.js`, `routes.js`

Routes:

* `GET /todos?limit=&offset=` — list user todos (requires auth)
* `GET /todos/:id` — get single todo (auth + owner check)
* `POST /todos` — create
* `PATCH /todos/:id` — update
* `DELETE /todos/:id` — delete

Controller responsibilities:

* Validate input via schema
* Call service functions
* Return JSON responses with proper HTTP codes
* Handle errors: return 400 for validation, 401 for missing/invalid token (handled by middleware), 403 if trying to access other user's todo, 404 for not found, 500 for server errors (log internally).

Example response shapes:

```json
// list
{ "rows": [ { "id": 1, "title": "Buy milk", "description": "Decrypted...", "completed": false, "created_at": "..."} ], "limit": 20, "offset": 0 }

// single
{ "id": 1, "title": "Buy milk", "description": "Decrypted...", "completed": false }
```

---

## Security & middleware

* Ensure `authMiddleware` runs before the todos router (or mount router with `app.use('/todos', authMiddleware, todosRouter)`).
* Rate limit todo creation and update endpoints. Example: max 30 create requests per minute per IP/user.
* Request body size limit: configure express `express.json({ limit: '10kb' })` or similar.
* Use `helmet()` globally for headers.
* Do not log decrypted descriptions or tokens.

---

## Tests: `tests/todos.test.js`

Write tests using Jest (or project test runner) covering:

1. **Create todo (happy path)**:

   * Authenticated user can create todo; database stores encrypted description; response returns decrypted description.
2. **Get todos pagination**:

   * Create multiple todos and test `limit` & `offset`.
3. **Owner isolation**:

   * User A cannot access User B's todos (expect 403 or 404).
4. **Update todo (change description)**:

   * Ensure new description stored encrypted and returned decrypted.
5. **Delete todo**:

   * Ensure deleting removes record; subsequent GET returns 404.
6. **Validation errors**:

   * Missing title → 400; description too long → 400.
7. **Rate limit behavior** (optional integration test):

   * Exceed create limit → expect 429.

Test setup:

* Use a test database (separate `DATABASE_URL_TEST`) and run migration before tests and clear tables after each test. Use transactions if possible.

---

## Example SQL queries (pg)

**Create**

```js
const result = await db.query(
  `INSERT INTO todos (user_id, title, description_encrypted) VALUES ($1, $2, $3) RETURNING id, title, completed, created_at`,
  [userId, title, descriptionEncrypted]
);
```

**Read list**

```js
const res = await db.query(
  `SELECT id, title, description_encrypted, completed, created_at FROM todos WHERE user_id = $1 ORDER BY created_at DESC LIMIT $2 OFFSET $3`,
  [userId, limit, offset]
);
```

**Read single**

```js
const res = await db.query(
  `SELECT id, title, description_encrypted, completed FROM todos WHERE id = $1 AND user_id = $2`,
  [todoId, userId]
);
```

**Update**

```js
-- dynamically build SET clause depending on provided fields but still use parameterized parameters
```

**Delete**

```js
await db.query('DELETE FROM todos WHERE id = $1 AND user_id = $2', [todoId, userId]);
```

---

## Error handling & logging

* Use a centralized error handler middleware to format errors.
* Log only error metadata — never include JWTs or plaintext descriptions.
* For DB errors, map unique constraint / foreign key errors to 4xx as appropriate.

---

## Acceptance criteria (how to mark task done)

1. All endpoints implemented and pass provided tests.
2. Todos descriptions stored as ciphertext in DB (verify by checking DB column directly).
3. API returns decrypted description to authenticated owner only.
4. Attempts to access another user’s todo return 403 or 404.
5. Input validation prevents invalid bodies.
6. Rate limiting returns 429 when threshold exceeded.
7. No sensitive values are logged.

---

