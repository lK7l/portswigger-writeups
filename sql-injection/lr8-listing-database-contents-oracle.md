# SQL injection attack: listing the database contents on Oracle

## Goal
- Determine the name of the table containing user data and the columns it contains, then extract the table contents to retrieve usernames and passwords for all users.
- Log in as the `administrator` user.

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
9. Inject a UNION query to retrieve table names: `' UNION SELECT table_name, NULL FROM all_tables--`.
10. Identify a table that likely contains user data by its name (in this case: `USERS_UBERMT`).
11. Inject a UNION query to retrieve column names for the `USERS_UBERMT` table: `' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_UBERMT'--`.
12. Inject a UNION query to retrieve username/password pairs: `' UNION SELECT USERNAME_JCEGKQ, PASSWORD_DCASED FROM USERS_UBERMT--`.
13. Locate the credentials for the `administrator` account in the response and log in with them.
14. Verify that the lab is marked as solved on the website.

## Result
- A list of all tables in the database is displayed.
- Column names for the `USERS_UBERMT` table are displayed.
- Username and password pairs are displayed on the web page.
- Successful authentication as the `administrator` user.

## Why it works
The application uses the `category` input in a SQL query (for example: `SELECT name, description FROM products WHERE category = 'Accessories'`) and reflects the results in the HTTP response, allowing a UNION-based injection to append rows from Oracle system views (`all_tables`, `all_tab_columns`) and user tables to the original result set.

## Fix
- Use prepared statements / parameterized queries so user input cannot change the structure of SQL queries.
- Use an allow-list for `category` values as an additional defense-in-depth control.
- Apply the principle of least privilege to database accounts to limit access to system views.
