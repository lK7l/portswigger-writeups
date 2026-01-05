# SQL injection UNION attack: retrieving data from other tables

## Goal
Perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.

## Vulnerable parameter
`GET /filter?category=...` (the `category` parameter).

## Steps to reproduce
1. On the lab website, select a product category (I used `Accessories`).
2. Open Burp Suite.
3. Go to `Proxy` → `HTTP history`.
4. Find the request with method `GET` and path `/filter?category=...`.
5. Right-click the request and select `Send to Repeater`.
6. Go to the `Repeater` tab.
7. Append the payload `' ORDER BY 1--` to the category value.
8. Re-send the request while incrementing the number (`ORDER BY 1`, `ORDER BY 2`, ...) and observe the response. When the server returns an error (e.g., HTTP 500), the previous number indicates the column count.
9. Inject a UNION query to retrieve username/password pairs: `' UNION SELECT username, password FROM users--`.
10. Locate the credentials for the `administrator` account in the response and log in with them.
11. Verify that the lab is marked as solved on the website.

## Result
- Username and password pairs are displayed on the web page.
- Successful authentication as the `administrator` user.

## Why it works
The application uses the `category` input in a SQL query (for example: `SELECT name, description FROM products WHERE category = 'Accessories'`) and reflects the results in the HTTP response, allowing a UNION-based injection to append rows from other tables (such as `users`) to the original result set.

## Fix
- Use prepared statements / parameterized queries so user input cannot change the structure of SQL queries.
- Use an allow-list for `category` values as an additional defense-in-depth control.
- Avoid storing plaintext passwords; use strong hashing algorithms (bcrypt, Argon2) with salts.
