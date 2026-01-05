# SQL injection attack: querying the database type and version on MySQL and Microsoft

## Goal
Display the database version string. [web:426]

## Vulnerable parameter
`GET /filter?category=...` (the `category` parameter). [web:426]

## Steps to reproduce
1. On the lab website, select a product category (I used `Accessories`). [web:426]
2. Open Burp Suite.
3. Go to `Proxy` → `HTTP history`.
4. Find the request with method `GET` and path `/filter?category=...`. [web:426]
5. Right-click the request and select `Send to Repeater`.
6. Go to the `Repeater` tab.
7. Append the payload `' ORDER BY 1--` to the category value. [web:126]
8. Re-send the request while incrementing the number (`ORDER BY 1`, `ORDER BY 2`, `ORDER BY 3`, ...) and observe the response. When the index exceeds the number of columns returned by the original query, the database returns an error; the previous number indicates the column count. [web:126]
9. Inject a UNION query that returns the DB version for MySQL/Microsoft (for example, using `SELECT @@version`) so it is reflected in the response. [web:410]
10. Verify that the database version string is displayed on the web page. [web:426]

## Result
The database version string is displayed on the web page. [web:426]

## Why it works
The application uses the `category` input in a SQL query and reflects the results in the HTTP response, allowing a UNION-based injection to append additional rows to the original result set. [web:426][web:126]

## Fix
- Use prepared statements / parameterized queries so user input cannot change the structure of SQL queries. [web:207]
- Use an allow-list for `category` values as an additional defense-in-depth control. [web:207]
