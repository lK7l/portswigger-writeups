# SQL injection attack: querying the database type and version on Oracle

## Goal
Display the database version string. [web:408]

## Vulnerable parameter
`GET /filter?category=...` (the `category` parameter). [web:408]

## Steps to reproduce
1. On the lab website, select a product category (I used `Accessories`). [web:408]
2. Open Burp Suite.
3. Go to `Proxy` → `HTTP history`.
4. Find the request with method `GET` and path `/filter?category=...`. [web:408]
5. Right-click the request and select `Send to Repeater`.
6. Go to the `Repeater` tab.
7. Append the payload `' ORDER BY 1--` to the category value. [web:126]
8. Re-send the request while incrementing the number (`ORDER BY 1`, `ORDER BY 2`, `ORDER BY 3`, ...) and observe the response. When the index exceeds the number of columns returned by the original query, the database returns an error; the previous number indicates the column count. [web:126]
9. Use a UNION query against Oracle to return the version banner from `v$version` (for example, selecting the `banner` column). [web:87][web:410]
10. Verify that the database version string is displayed in the HTTP response / web page. [web:408]

## Result
The database version string is displayed on the web page. [web:408]

## Why it works
The application uses the `category` input in a SQL query and reflects the query results in the response, so a UNION-based injection can append additional rows to the original result set. [web:408][web:126]

## Fix
- Use prepared statements / parameterized queries so user input cannot change the structure of SQL queries. [web:207]
- Use an allow-list for `category` values as an additional defense-in-depth control. [web:207]
