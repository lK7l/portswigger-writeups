# Blind SQL injection with conditional errors

## Goal
- Use a blind SQL injection technique to determine the `administrator` user's password.
- Log in to the application as `administrator`.

## Vulnerable parameter
`Cookie: TrackingId=...` (tracking cookie value).

## Steps to reproduce (no payloads)
1. Open Burp Suite.
2. On the lab website, open any product page to generate a GET request.
3. Go to `Proxy` → `HTTP history`.
4. Find a request that uses the `GET` method.
5. Right-click the request → `Send to Repeater`.
6. In `Repeater`, locate the `Cookie` header and the `TrackingId` parameter.
7. Confirm that modifying `TrackingId` causes a detectable server-side difference when a SQL error occurs (for example, the application returns a custom error / HTTP 500 when the SQL query errors).
8. Use a conditional expression that triggers a database error only when the tested condition is true, in order to:
   - determine the password length (iterate candidate lengths and compare the server response);
   - determine each password character (iterate the position and candidate characters, compare the server response).
9. To automate the iteration, send the request to `Intruder`, mark the payload position, set the character set, and start the attack.
10. Log in as `administrator` using the recovered password.

## Result
Successful authentication as `administrator` (after extracting the password by observing differences in server responses).

## Why it works
The application uses the `TrackingId` value inside a SQL query, but it does not return the query result to the user and the responses look the same unless a SQL error occurs.
An attacker can craft a condition that intentionally triggers an error (for example, a divide-by-zero) only when the tested statement is true, and then infer true/false based on response differences (HTTP 500 on error vs HTTP 200 normally).
By repeating this test for different lengths and character guesses, the password can be reconstructed one character at a time without any direct SQL output.

## How to fix
- Use **prepared statements / parameterized queries** for all database access to ensure user input is treated as data, not executable SQL.
- Apply allow-list validation: enforce that `TrackingId` matches an expected format (for example, UUID or fixed-length token from an allowed character set) and reject anything else.
- Avoid dynamic SQL string concatenation driven by cookies/user input; store and query tracking identifiers as plain data via parameters.
- Reduce information leakage via errors: do not return distinguishable error messages/status codes based on SQL exceptions; log details server-side and return a generic response to the client.
- Enforce least privilege for the database user: the application account should have only the minimal permissions required, to limit impact even if injection occurs.
