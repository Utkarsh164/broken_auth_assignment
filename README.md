# The Silent Server (Backend Debugging Assignment)

This API is intentionally broken. Your task is to debug it and complete the authentication flow.

## Setup

1. Install dependencies:
   ```bash
   npm install
   ```

2. Start the server:
   ```bash
   npm start
   ```
   Server runs at: `http://localhost:3000`

## Assignment Objective

The goal is to fix the broken authentication endpoints so that a user can:
1.  **Login** to get a session ID and OTP.
2.  **Verify the OTP** to get a valid session cookie.
3.  **Exchange the Session** for a JWT Access Token.
4.  **Access Protected Routes** using the token.

You will need to use your browser's developer tools, network inspection, and server logs to debug.

---

## Tasks & Verification

### Task 1: Fix Login
**Endpoint:** `POST /auth/login`
The server should generate a session and log an OTP to the console.

**Test Command:**
```bash
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"<YOUR_EMAIL@example.com>","password":"password123"}'
```
**Expected Outcome:**
- Server logs the OTP (e.g., `[OTP] Session abc12345 generated`).
- Response contains `loginSessionId`.

### Task 2: Fix OTP Verification
**Endpoint:** `POST /auth/verify-otp`
The server fails to verify the OTP correctly. You need to find out why.
*Hint: Check data types and how cookies are set.*

**Test Command:**
(Replace `<loginSessionId>` and `<otp>` with values from Task 1)
```bash
curl -c cookies.txt -X POST http://localhost:3000/auth/verify-otp \
  -H "Content-Type: application/json" \
  -d '{"loginSessionId":"<loginSessionId>","otp":"<otp_from_logs>"}'
```
**Expected Outcome:**
- `cookies.txt` is created containing a session cookie.
- Response says "OTP verified".

### Task 3: Fix Token Generation
**Endpoint:** `POST /auth/token`
This endpoint is supposed to issue a JWT, but it has a bug in how it reads the session.

**Test Command:**
```bash
# Uses the cookie captured in Task 2
curl -b cookies.txt -X POST http://localhost:3000/auth/token
```
**Expected Outcome:**
- Response contains `{ "access_token": "..." }`.

### Task 4: Fix Protected Route Access
**Endpoint:** `GET /protected`
Ensure the middleware correctly validates the token.

**Test Command:**
```bash
# Replace <jwt> with the token from Task 3
curl -H "Authorization: Bearer <jwt>" http://localhost:3000/protected
```
**Expected Outcome:**
- Response: `{ "message": "Access granted", "user": ... }`

---


## Expected Output

After fixing the bugs, you should be able to run the following sequence successfully:

1.  **Login**: Receive a `loginSessionId` and see an OTP in the server logs.
2.  **Verify OTP**: Receive a session cookie (`session_token`).
3.  **Get Token**: Exchange the session cookie for a JWT (`access_token`).
4.  **Access Protected Route**: Use the JWT to get a 200 OK response with user details and a **unique Success Flag**.

**Important**: You must use **your own email address** when testing the login flow. The success flag is generated based on the email you use.




## Submission

To submit your assignment:

1.  Push your code to a **Public GitHub Repository**.
2.  Add a file named `output.txt` in your repository.
    *   This file must contain the terminal output of all 4 test commands (Login, Verify OTP, Get Token, Access Protected Route).
    *   Ensure the final command's output showing the `success_flag` is clearly visible in this file.
3.  Share the link to your repository.

---

## My Output

### Command 1 — Login
```bash
curl -X POST http://localhost:3000/auth/login -H "Content-Type: application/json" -d '{"email":"utkarsh.srawat2003@gmail.com","password":"password123"}'
```
**Response:**
```json
{"message":"OTP sent","loginSessionId":"mvx7a"}
```

---

### Command 2 — Verify OTP
```bash
curl -c cookies.txt -X POST http://localhost:3000/auth/verify-otp -H "Content-Type: application/json" -d '{"loginSessionId":"mvx7a","otp":"187206"}'
```
**Response:**
```json
{"message":"OTP verified","sessionId":"mvx7a"}
```

---

### Command 3 — Get Token
```bash
curl -b cookies.txt -X POST http://localhost:3000/auth/token
```
**Response:**
```json
{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InV0a2Fyc2guc3Jhd2F0MjAwM0BnbWFpbC5jb20iLCJzZXNzaW9uSWQiOiJtdng3YSIsImlhdCI6MTc3MDg5MDI5MiwiZXhwIjoxNzcwODkxMTkyfQ.l2PtQHFr1GQIlp8yKkVJyK2FSw0rVXZTUP-m5Nvg8Us","expires_in":900}
```

---

### Command 4 — Access Protected Route
```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InV0a2Fyc2guc3Jhd2F0MjAwM0BnbWFpbC5jb20iLCJzZXNzaW9uSWQiOiJtdng3YSIsImlhdCI6MTc3MDg5MDI5MiwiZXhwIjoxNzcwODkxMTkyfQ.l2PtQHFr1GQIlp8yKkVJyK2FSw0rVXZTUP-m5Nvg8Us" http://localhost:3000/protected
```
**Response:**
```json
{"message":"Access granted","user":{"email":"utkarsh.srawat2003@gmail.com","sessionId":"mvx7a","iat":1770890292,"exp":1770891192},"success_flag":"FLAG-dXRrYXJzaC5zcmF3YXQyMDAzQGdtYWlsLmNvbV9DT01QTEVURURfQVNTSUdOTUVOVA=="}
```

---

### Server Logs
```
Server running at http://localhost:3000
POST /auth/login
[OTP] Session mvx7a generated : 187206
POST /auth/login -> 200 (71ms)
POST /auth/verify-otp
POST /auth/verify-otp -> 200 (25ms)
POST /auth/token
POST /auth/token -> 200 (11ms)
GET /protected
GET /protected -> 200 (5ms)
```
