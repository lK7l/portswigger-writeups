# Visible error-based SQL injection

## Goal
- Extract the password for `administrator` via visible (error-based) SQL injection.
- Log in as `administrator`.

## Vulnerable parameter
`Cookie: TrackingId=...` (tracking cookie value).

## Steps to reproduce (no payloads)
1. Open Burp Suite.
2. On the test site, open any product page to generate a `GET` request.
3. Go to `Proxy` → `HTTP history`.
4. Find the request with method `GET` to the application.
5. Right-click the request → `Send to Repeater`.
6. In `Repeater`, locate the `Cookie` header and the `TrackingId` parameter.
7. Replace the `TrackingId` value with input that manipulates the backend SQL query and forces a database type-conversion error so that the server returns a verbose database error message containing data from a subquery.  
   Payload: `REDACTED`.
8. Observe that the server response includes a verbose database/SQL error.
9. The error message discloses sensitive data (administrator password) via error text.
10. Log in as `administrator` using the recovered password.

## Result
Successful authentication as `administrator` after extracting the password via error message disclosure.

## Why this works
The application uses the `TrackingId` cookie value in a server-side SQL query (e.g., for analytics), and the query is executed by the database even though its results are not directly rendered in the page.
SQL injection occurs when dynamic SQL is built by concatenating user-controlled input (including cookies), allowing an attacker to change the query logic.
Visible (error-based) data exfiltration is possible because the application returns verbose database/SQL error messages to the client; such errors may disclose parts of the query and sensitive data included in exception output (information disclosure via error messages).

## How to fix
- Remove string concatenation and use parameterized queries (prepared statements) and/or properly implemented stored procedures so the database treats user input as data, not executable SQL.
- Apply allow-list validation for `TrackingId` based on the expected format (e.g., UUID / strict regex) and reject any value that does not match.
- Disable verbose database errors in production: return a generic error to users and keep detailed diagnostics only in internal logs.
- Ensure the database user used by the application follows the principle of least privilege.
