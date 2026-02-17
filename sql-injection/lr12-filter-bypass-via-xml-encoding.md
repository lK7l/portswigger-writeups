# SQL injection vulnerability in a WHERE clause allowing retrieval of hidden data

## Goal
Make unreleased (hidden) products visible.

## Vulnerable parameter
`GET /filter?category=...` (the `category` parameter).

## Steps to reproduce
1. On the lab website, select a product category (I used `Accessories`).
2. Open Burp Suite.
3. Go to `Proxy` → `HTTP history`.
4. Find the request with method `GET` and path `/filter?category=...`.
5. Right-click the request and select `Send to Repeater`.
6. Go to the `Repeater` tab.
7. In the request parameters, append the following SQL payload after the category value (in my case after `Accessories`): `' OR 1=1--`.
8. Click `Send`.
9. Check the lab website to confirm the result.

## Result
Unreleased (hidden) products become visible. [web:78]

## Why it works
The application runs a SQL query similar to: `SELECT * FROM products WHERE category = 'Gifts' AND released = 1`. [web:78]  
The `'` closes the string, `OR 1=1` adds an always-true condition, and `--` comments out the rest of the original query (such as `AND released = 1`), so the filter returns unreleased products too. [web:78][web:82][web:126]  
In some databases (for example MySQL), `--` must be followed by a space to be treated as a comment. [web:126]

## Fix
Use prepared statements / parameterized queries so user input is treated as data, not executable SQL. [web:207]
