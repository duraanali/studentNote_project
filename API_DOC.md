# Student Records System API Documentation

## Base URL

```
http://localhost:3000
```

All endpoints below are relative to this base URL.

---

## Important: Authentication (JWT)

Most endpoints require a JWT access token.

### How to send the token (Postman)

If you have not built a frontend yet, use **Postman** to register, log in, and call the API.

1. Open Postman and create a new request.
2. Set the method to **POST** and the URL to `http://localhost:3000/api/auth/register` (or `/api/auth/login` if you already have an account).
3. Go to the **Body** tab → select **raw** → choose **JSON** → send your register/login JSON.
4. Send the request and copy the `token` value from the response.
5. For protected endpoints (profile, notes), create a new request and open the **Authorization** tab.
6. Set **Type** to **Bearer Token** and paste your token into the **Token** field.

Alternatively, you can add it manually under the **Headers** tab:

| Header          | Value                  |
| --------------- | ---------------------- |
| `Authorization` | `Bearer <your_token>`  |

Example:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Token details

- Tokens expire after **24 hours**.
- After expiry, log in again to get a new token.
- Protected routes will reject requests with no token, a bad token, or an expired token.

### Content-Type

For endpoints that send a JSON body (register, login, create note, update note), set:

| Header         | Value            |
| -------------- | ---------------- |
| `Content-Type` | `application/json` |

In Postman: Body → **raw** → **JSON**.

---

## Auth Endpoints

### 1. Register Student

Creates a new student account and returns a JWT token.

| Item        | Value                    |
| ----------- | ------------------------ |
| **Method**  | `POST`                   |
| **URL**     | `/api/auth/register`     |
| **Auth**    | Not required             |

#### Request Body

| Field      | Type     | Required | Description              |
| ---------- | -------- | -------- | ------------------------ |
| `name`     | string   | Yes      | Student's full name      |
| `email`    | string   | Yes      | Unique email address     |
| `password` | string   | Yes      | Account password         |
| `age`      | number   | No       | Student's age            |
| `grade`    | string   | No       | Grade level              |
| `major`    | string   | No       | Field of study           |

#### Example Request Body

```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "securePassword123",
  "age": 20,
  "grade": "Sophomore",
  "major": "Computer Science"
}
```

#### Success Response — `201 Created`

```json
{
  "success": true,
  "message": "Student registered successfully",
  "data": {
    "student": {
      "id": "clxyz123abc",
      "name": "Jane Doe",
      "email": "jane@example.com",
      "age": 20,
      "grade": "Sophomore",
      "major": "Computer Science",
      "createdAt": "2026-11-08T18:00:00.000Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

Save the `token` — you will need it for protected endpoints.

#### Error Responses

**`400 Bad Request` — Missing required fields**

Meaning: `name`, `email`, or `password` was not provided.

```json
{
  "success": false,
  "message": "Name, email, and password are required"
}
```

**`400 Bad Request` — Email already exists**

Meaning: A student with that email is already registered. Use a different email or log in instead.

```json
{
  "success": false,
  "message": "Student with this email already exists"
}
```

**`500 Internal Server Error` — Server failure**

Meaning: Something went wrong on the server (database issue, unexpected error, etc.).

```json
{
  "success": false,
  "message": "Error registering student",
  "error": "Error details here"
}
```

---

### 2. Login Student

Logs in an existing student and returns a JWT token.

| Item        | Value                |
| ----------- | -------------------- |
| **Method**  | `POST`               |
| **URL**     | `/api/auth/login`    |
| **Auth**    | Not required         |

#### Request Body

| Field      | Type   | Required | Description          |
| ---------- | ------ | -------- | -------------------- |
| `email`    | string | Yes      | Registered email     |
| `password` | string | Yes      | Account password     |

#### Example Request Body

```json
{
  "email": "jane@example.com",
  "password": "securePassword123"
}
```

#### Success Response — `200 OK`

```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "student": {
      "id": "clxyz123abc",
      "name": "Jane Doe",
      "email": "jane@example.com",
      "age": 20,
      "grade": "Sophomore",
      "major": "Computer Science",
      "createdAt": "2026-11-08T18:00:00.000Z",
      "updatedAt": "2026-11-08T18:00:00.000Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### Error Responses

**`400 Bad Request` — Missing email or password**

Meaning: One or both required login fields were omitted.

```json
{
  "success": false,
  "message": "Email and password are required"
}
```

**`401 Unauthorized` — Invalid credentials**

Meaning: The email does not exist, or the password is wrong. The API returns the same message in both cases for security.

```json
{
  "success": false,
  "message": "Invalid email or password"
}
```

**`500 Internal Server Error` — Server failure**

```json
{
  "success": false,
  "message": "Error during login",
  "error": "Error details here"
}
```

---

### 3. Get Student Profile

Returns the currently authenticated student's profile.

| Item        | Value                  |
| ----------- | ---------------------- |
| **Method**  | `GET`                  |
| **URL**     | `/api/auth/profile`    |
| **Auth**    | Required (Bearer JWT)  |
| **Body**    | None                   |

#### Headers

| Header          | Required | Value                 |
| --------------- | -------- | --------------------- |
| `Authorization` | Yes      | `Bearer <your_token>` |

#### Success Response — `200 OK`

```json
{
  "success": true,
  "message": "Profile retrieved successfully",
  "data": {
    "student": {
      "id": "clxyz123abc",
      "name": "Jane Doe",
      "email": "jane@example.com",
      "age": 20,
      "grade": "Sophomore",
      "major": "Computer Science"
    }
  }
}
```

#### Error Responses

See [Common Auth Errors](#common-auth-errors) below for `401` and `403`.

**`500 Internal Server Error`**

```json
{
  "success": false,
  "message": "Error retrieving profile",
  "error": "Error details here"
}
```

---

## Notes Endpoints

All notes endpoints require authentication. Notes are scoped to the logged-in student — you can only access your own notes.

### Common Auth Errors (all protected notes routes)

These can occur on any notes endpoint if the token is missing or invalid:

**`401 Unauthorized` — No token**

Meaning: The `Authorization` header was missing, or did not include a Bearer token.

```json
{
  "success": false,
  "message": "Access token required"
}
```

**`401 Unauthorized` — Student not found**

Meaning: The token is valid, but the student ID inside it no longer exists in the database.

```json
{
  "success": false,
  "message": "Invalid token - student not found"
}
```

**`403 Forbidden` — Invalid or expired token**

Meaning: The token is malformed, tampered with, or past its 24-hour expiry. Log in again to get a new token.

```json
{
  "success": false,
  "message": "Invalid or expired token"
}
```

---

### 4. Get All Notes

Returns all notes belonging to the authenticated student, newest first.

| Item        | Value                  |
| ----------- | ---------------------- |
| **Method**  | `GET`                  |
| **URL**     | `/api/notes`           |
| **Auth**    | Required (Bearer JWT)  |
| **Body**    | None                   |

#### Headers

| Header          | Required | Value                 |
| --------------- | -------- | --------------------- |
| `Authorization` | Yes      | `Bearer <your_token>` |

#### Success Response — `200 OK`

```json
{
  "success": true,
  "message": "Notes retrieved successfully",
  "data": [
    {
      "id": "clnote456def",
      "title": "Study Plan",
      "content": "Review chapters 1-3 before the exam.",
      "studentId": "clxyz123abc",
      "createdAt": "2026-11-08T19:00:00.000Z",
      "updatedAt": "2026-11-08T19:00:00.000Z"
    }
  ]
}
```

If the student has no notes, `data` will be an empty array: `[]`.

#### Error Responses

Auth errors: see [Common Auth Errors](#common-auth-errors).

**`500 Internal Server Error`**

```json
{
  "success": false,
  "message": "Error retrieving notes",
  "error": "Error details here"
}
```

---

### 5. Get a Single Note

Returns one note by ID, only if it belongs to the authenticated student.

| Item        | Value                     |
| ----------- | ------------------------- |
| **Method**  | `GET`                     |
| **URL**     | `/api/notes/:id`          |
| **Auth**    | Required (Bearer JWT)     |
| **Body**    | None                      |

#### URL Parameters

| Param | Type   | Required | Description                          |
| ----- | ------ | -------- | ------------------------------------ |
| `id`  | string | Yes      | The note ID (e.g. `clnote456def`)    |

Example URL: `/api/notes/clnote456def`

#### Headers

| Header          | Required | Value                 |
| --------------- | -------- | --------------------- |
| `Authorization` | Yes      | `Bearer <your_token>` |

#### Success Response — `200 OK`

```json
{
  "success": true,
  "message": "Note retrieved successfully",
  "data": {
    "id": "clnote456def",
    "title": "Study Plan",
    "content": "Review chapters 1-3 before the exam.",
    "studentId": "clxyz123abc",
    "createdAt": "2026-11-08T19:00:00.000Z",
    "updatedAt": "2026-11-08T19:00:00.000Z"
  }
}
```

#### Error Responses

Auth errors: see [Common Auth Errors](#common-auth-errors).

**`404 Not Found` — Note does not exist (or is not yours)**

Meaning: No note with that ID was found for the logged-in student. Wrong ID, deleted note, or another student's note all return this.

```json
{
  "success": false,
  "message": "Note not found"
}
```

**`500 Internal Server Error`**

```json
{
  "success": false,
  "message": "Error retrieving note",
  "error": "Error details here"
}
```

---

### 6. Create a Note

Creates a new note for the authenticated student.

| Item        | Value                  |
| ----------- | ---------------------- |
| **Method**  | `POST`                 |
| **URL**     | `/api/notes`           |
| **Auth**    | Required (Bearer JWT)  |

#### Headers

| Header          | Required | Value                  |
| --------------- | -------- | ---------------------- |
| `Authorization` | Yes      | `Bearer <your_token>`  |
| `Content-Type`  | Yes      | `application/json`     |

#### Request Body

| Field     | Type   | Required | Description        |
| --------- | ------ | -------- | ------------------ |
| `title`   | string | Yes      | Note title         |
| `content` | string | Yes      | Note body text     |

#### Example Request Body

```json
{
  "title": "Study Plan",
  "content": "Review chapters 1-3 before the exam."
}
```

#### Success Response — `201 Created`

```json
{
  "success": true,
  "message": "Note created successfully",
  "data": {
    "id": "clnote456def",
    "title": "Study Plan",
    "content": "Review chapters 1-3 before the exam.",
    "studentId": "clxyz123abc",
    "createdAt": "2026-11-08T19:00:00.000Z",
    "updatedAt": "2026-11-08T19:00:00.000Z"
  }
}
```

#### Error Responses

Auth errors: see [Common Auth Errors](#common-auth-errors).

**`400 Bad Request` — Missing title or content**

Meaning: One or both required fields were missing or empty.

```json
{
  "success": false,
  "message": "Title and content are required"
}
```

**`500 Internal Server Error`**

```json
{
  "success": false,
  "message": "Error creating note",
  "error": "Error details here"
}
```

---

### 7. Update a Note

Replaces the title and content of an existing note owned by the authenticated student.

| Item        | Value                  |
| ----------- | ---------------------- |
| **Method**  | `PUT`                  |
| **URL**     | `/api/notes/:id`       |
| **Auth**    | Required (Bearer JWT)  |

#### URL Parameters

| Param | Type   | Required | Description     |
| ----- | ------ | -------- | --------------- |
| `id`  | string | Yes      | The note ID     |

Example URL: `/api/notes/clnote456def`

#### Headers

| Header          | Required | Value                  |
| --------------- | -------- | ---------------------- |
| `Authorization` | Yes      | `Bearer <your_token>`  |
| `Content-Type`  | Yes      | `application/json`     |

#### Request Body

| Field     | Type   | Required | Description              |
| --------- | ------ | -------- | ------------------------ |
| `title`   | string | Yes      | Updated note title       |
| `content` | string | Yes      | Updated note body text   |

Both fields are required on every update (full replace of title and content).

#### Example Request Body

```json
{
  "title": "Updated Study Plan",
  "content": "Review chapters 1-5 and practice problems."
}
```

#### Success Response — `200 OK`

```json
{
  "success": true,
  "message": "Note updated successfully",
  "data": {
    "id": "clnote456def",
    "title": "Updated Study Plan",
    "content": "Review chapters 1-5 and practice problems.",
    "studentId": "clxyz123abc",
    "createdAt": "2026-11-08T19:00:00.000Z",
    "updatedAt": "2026-11-08T20:00:00.000Z"
  }
}
```

#### Error Responses

Auth errors: see [Common Auth Errors](#common-auth-errors).

**`400 Bad Request` — Missing title or content**

```json
{
  "success": false,
  "message": "Title and content are required"
}
```

**`404 Not Found` — Note does not exist (or is not yours)**

```json
{
  "success": false,
  "message": "Note not found"
}
```

**`500 Internal Server Error`**

```json
{
  "success": false,
  "message": "Error updating note",
  "error": "Error details here"
}
```

---

### 8. Delete a Note

Deletes a note owned by the authenticated student.

| Item        | Value                  |
| ----------- | ---------------------- |
| **Method**  | `DELETE`               |
| **URL**     | `/api/notes/:id`       |
| **Auth**    | Required (Bearer JWT)  |
| **Body**    | None                   |

#### URL Parameters

| Param | Type   | Required | Description     |
| ----- | ------ | -------- | --------------- |
| `id`  | string | Yes      | The note ID     |

Example URL: `/api/notes/clnote456def`

#### Headers

| Header          | Required | Value                 |
| --------------- | -------- | --------------------- |
| `Authorization` | Yes      | `Bearer <your_token>` |

#### Success Response — `204 No Content`

```json
{
  "success": true,
  "message": "Note deleted successfully"
}
```

#### Error Responses

Auth errors: see [Common Auth Errors](#common-auth-errors).

**`404 Not Found` — Note does not exist (or is not yours)**

```json
{
  "success": false,
  "message": "Note not found"
}
```

**`500 Internal Server Error`**

```json
{
  "success": false,
  "message": "Error deleting note",
  "error": "Error details here"
}
```

---

## Other Responses

### Root / Health Check

| Item       | Value |
| ---------- | ----- |
| **Method** | `GET` |
| **URL**    | `/`   |
| **Auth**   | None  |

Returns basic API info (`200 OK`).

### Unknown Route — `404 Not Found`

Any URL that does not match a defined route:

```json
{
  "success": false,
  "message": "Route not found"
}
```

### Unexpected Server Error — `500 Internal Server Error`

Unhandled errors elsewhere in the app:

```json
{
  "success": false,
  "message": "Something went wrong!",
  "error": {}
}
```

---

## Quick Reference

| Method   | Endpoint             | Auth | Description              |
| -------- | -------------------- | ---- | ------------------------ |
| `POST`   | `/api/auth/register` | No   | Create student account   |
| `POST`   | `/api/auth/login`    | No   | Log in and get token     |
| `GET`    | `/api/auth/profile`  | Yes  | Get current profile      |
| `GET`    | `/api/notes`         | Yes  | List all your notes      |
| `GET`    | `/api/notes/:id`     | Yes  | Get one note             |
| `POST`   | `/api/notes`         | Yes  | Create a note            |
| `PUT`    | `/api/notes/:id`     | Yes  | Update a note            |
| `DELETE` | `/api/notes/:id`     | Yes  | Delete a note            |

---

## Suggested Testing Order (Postman)

1. **Register** a student → copy the `token` from the response.
2. **Login** with the same email/password → confirm you get a new token.
3. Set the `Authorization` header to `Bearer <token>` for the remaining requests.
4. **Create** a note → copy the note `id`.
5. **Get all notes** → confirm the new note appears.
6. **Get one note** using that `id`.
7. **Update** the note.
8. **Delete** the note.
9. Try getting the deleted note again → expect `404`.
10. Try a notes request **without** a token → expect `401`.
