# Blind SQL Injection with Time Delays

## Objective
- Cause a 10-second delay in page loading.

## Vulnerable Parameter
`Cookie: TrackingId=...` (value of tracking cookie).

## Steps to Reproduce
1. Open Burp Suite.
2. Open a product card on the test page to generate a GET request.
3. Open `Proxy` → `HTTP history`.
4. Find the request with method `GET` to the application.
5. Right-click (RMB) on the found request → `Send to Repeater`.
6. In `Repeater`, find the `Cookie` header and `TrackingId` parameter.
7. Remove the body of 'TrackingId' and write the request `x'||pg_sleep(10)--`.
8. A 10-second delay will appear.

## Result
Appearance of a 10-second delay in page loading.

## Why it works
- **Lack of Sanitization:** The application accepts the `Cookie` header value (`TrackingId` parameter) and directly inserts it into the SQL query to the database without validating or escaping special characters.
- **Code Interpretation:** The database perceives the passed string not as data (plain text), but as an executable command.
- **Synchronous Execution:** The web server waits for a response from the database before sending an HTTP response to the user. Since the DB "sleeps" for 10 seconds, the web server also delays the response, which is observed by the attacker.

## How to fix
- **Use of Parameterized Queries (Prepared Statements)**
This is the primary and most reliable method of protection. Instead of string concatenation, use placeholders (bind variables).
- **Input Validation**
Ensure that `TrackingId` matches the expected format before passing it to the query.
