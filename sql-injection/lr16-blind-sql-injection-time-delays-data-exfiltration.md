# Blind SQL Injection with Time Delays and Data Retrieval

## Objective
- Brute-force the password for 'administrator' using time-based delays.
- Log in to the system as 'administrator'.

## Vulnerable Parameter
`Cookie: TrackingId=...` (value of tracking cookie).

## Steps to Reproduce
1. Open Burp Suite.
2. Open a product card on the test page to generate a GET request.
3. Open `Proxy` → `HTTP history`.
4. Find the request with method `GET` to the application.
5. Right-click (RMB) on the found request → `Send to Repeater`.
6. In `Repeater`, find the `Cookie` header and `TrackingId` parameter.
7. Verify that changing `TrackingId` results in a distinguishable server response (e.g., a 10-second delay occurs under a true condition).
8. Use a conditional expression that triggers a delay only when the condition is true to:
   - Determine the password length (iterate through length values and compare server response).
   - Determine each character of the password (iterate through position and candidate characters, compare server response).
9. To automate the brute-force process, send the request to `Intruder`, mark the position for payload insertion, set the character set, and start the attack.
10. Log in as `administrator` using the obtained password.

## Result
Successful authorization under the `administrator` account (after extracting the password via server response differences).

## Why it works
- **Lack of Filtering:** The application does not validate the `TrackingId` content before using it in the SQL query, allowing arbitrary commands to be injected.
- **Synchronous Request Processing:** The web server waits for the SQL query execution to complete before sending a response.
- **Character-by-Character Brute-force:** An attacker can ask the database questions like "Is the first character of the password 'a'?" and receive a "Yes" (delay) or "No" (immediate response). This allows reconstructing data bit by bit, even if the application returns no error messages or data.

## How to fix
- **Parameterized Queries (Prepared Statements)**
Use the parameterization mechanism (bind variables) in all SQL queries. This guarantees that the database will interpret `TrackingId` only as data, not as executable code.
- **Input Validation**
Check the `TrackingId` format (e.g., it must be a UUID or a fixed-length string of specific characters) before performing any database operations.
- **Configure Web Application Firewall (WAF)**
Set up WAF to block requests containing keywords for time-based injections (`pg_sleep`, `WAITFOR`, `BENCHMARK`).
