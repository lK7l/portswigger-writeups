# Blind SQL injection with conditional responses

## Goal
- Exploit a blind SQL injection vulnerability to extract the `administrator` user's password.
- Log in as the `administrator` user.

## Vulnerable parameter
`Cookie: TrackingId=...` (the `TrackingId` cookie value).

## Steps to reproduce
1. On the lab website, navigate to any page to generate HTTP requests.
2. Open Burp Suite.
3. Go to `Proxy` → `HTTP history`.
4. Find any request (e.g., `GET /`) and observe the `Cookie` header containing `TrackingId`.
5. Right-click the request and select `Send to Repeater`.
6. Go to the `Repeater` tab.
7. Inject a payload to determine the length of the administrator's password: modify the `TrackingId` cookie to:  
   `TrackingId=<original_value>' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a'--`  
   Increment the number in `LENGTH(password)>N` using binary search or iteration until the "Welcome back" message disappears, indicating the exact password length.
8. Extract the password character by character by modifying the `TrackingId` cookie:  
   `TrackingId=<original_value>' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)>'m'--`  
   Use binary search on ASCII values (e.g., `>'m'`, `<'m'`, `='m'`) to determine each character. Repeat for positions 2, 3, 4, etc., until the full password is extracted(vhew8c15890k5lsrt740).
9. Log in as `administrator` using the extracted password.
10. Verify that the lab is marked as solved on the website.

## Result
Successful authentication as the `administrator` user.

## Why it works
The application uses the `TrackingId` cookie value in a SQL query without proper parameterization (for example: `SELECT * FROM tracking WHERE id = '<TrackingId>'`). By injecting a conditional SQL statement, the attacker can infer information based on the application's behavior: when the injected condition is true, the application displays a "Welcome back" message; when false, the message does not appear. This boolean-based blind SQL injection allows extracting sensitive data (such as passwords) one character at a time by observing changes in the HTTP response.

## Fix
- Use prepared statements / parameterized queries so user input (including cookies) cannot alter the structure of SQL queries.
- Implement proper input validation and sanitization for all user-controlled data, including cookies.
- Apply the principle of least privilege to database accounts to limit access to sensitive tables.
- Use secure session identifiers that are not vulnerable to SQL injection (e.g., cryptographically random tokens stored separately from SQL queries).
- Implement rate limiting and anomaly detection to prevent automated password extraction attacks.
