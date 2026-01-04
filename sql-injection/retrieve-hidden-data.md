# SQL injection vulnerability allowing authentication bypass

## Goal
Log in to an account without knowing the correct username and password. [web:59]

## Vulnerable parameter
The `username` (login) parameter in the POST request to the login function is vulnerable because it is used in a SQL query without parameterization. [web:59][web:66]

## Steps to reproduce
1. On the lab website, click **My account**. [web:59]
2. In the **Login** field, enter: `administrator'--`. [web:59]
3. In the **Password** field, enter any value (to satisfy client-side validation). [web:59]
4. Click **Log in**.
5. Verify that you are logged in as the `administrator` user. [web:59]

## Result
Successful authentication as `administrator`. [web:59]

## Why it works
The server-side login query is typically something like: `SELECT * FROM users WHERE username = '<input>' AND password = '<input>'`. [web:66][web:57]  
The payload `administrator'--` closes the string after `administrator` and then starts a SQL comment with `--`, which causes the rest of the query (including the password check) to be ignored, so the application authenticates without validating the password. [web:66][web:57]

## Fix
- Use prepared statements / parameterized queries so user input is treated as data, not executable SQL. [web:207][web:208]
- Apply strict allow-list validation for usernames (as an extra defense-in-depth measure). [web:207]
