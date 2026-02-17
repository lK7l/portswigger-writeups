# SQL injection UNION attack: determining the number of columns returned by the query

## Goal
Make a value injected via a UNION query appear in the lab response. [web:126][web:310]

## Vulnerable parameter
`GET /filter?category=...` (the `category` parameter).

## Steps to reproduce
1. On the lab website, select a product category (I used `Accessories`).
2. Open Burp Suite.
3. Go to `Proxy` → `HTTP history`.
4. Find the request with method `GET` and path `/filter?category=...`.
5. Right-click the request and select `Send to Repeater`.
6. Go to the `Repeater` tab.
7. Append the SQL payload `' ORDER BY 1--` to the `category` value.
8. Repeat the request while incrementing the number (`ORDER BY 1`, `ORDER BY 2`, `ORDER BY 3`, ...) and observe the application response. When the index exceeds the number of columns in the result set, the database returns an error (in my case the previous number indicated the column count).
9. To identify which column can display text, test a UNION payload that places a string into each column in turn (for example: `' UNION SELECT NULL, 'null', NULL--`) until the injected text appears in the response without errors.
10. Verify the result on the lab website.

## Result
The injected text value appears on the web page.
## Why it works
The application uses the `category` input in a SQL query whose results are reflected in the HTTP response, so a UNION-based injection can append additional rows to the original result set.
By injecting a series of `ORDER BY <index>` payloads, it is possible to infer the number of columns, because `ORDER BY` can reference columns by position, and the database throws an error when the index is out of range.

## Fix
- Use prepared statements / parameterized queries so user input cannot change the structure of SQL queries.
- Use an allow-list for `category` values as an additional defense-in-depth control.
