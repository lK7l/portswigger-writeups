# SQL injection UNION attack: finding a column containing text

## Goal
Perform a SQL injection UNION attack that returns an additional row containing the provided value.

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
9. Inject a UNION query to return the provided random value by placing it in each column position in turn until it appears without errors: `' UNION SELECT NULL, 'PmPnJi', NULL--`.
10. Verify that the lab is marked as solved on the website.

## Result
The provided random value is displayed on the web page.

## Why it works
The application uses the `category` input in a SQL query (for example: `SELECT name, description FROM products WHERE category = 'Accessories'`) and reflects the results in the HTTP response, allowing a UNION-based injection to append additional rows. By testing which column position can display text (replacing `NULL` with a string literal in each position), it is possible to identify which columns have string-compatible data types and are rendered in the application response.

## Fix
- Use prepared statements / parameterized queries so user input cannot change the structure of SQL queries.
- Use an allow-list for `category` values as an additional defense-in-depth control.
