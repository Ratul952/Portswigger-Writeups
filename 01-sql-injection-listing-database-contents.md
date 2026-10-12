Lab name: SQL injection attack, listing the database contents on non-Oracle databases
Category: SQL Injection
Difficulty: Practitioner

The vulnerability: The site’s product filter takes a category parameter and puts it straight into a SQL query without sanitizing it. That lets an attacker inject their own SQL instead of just a category name.

How I found it: I intercepted the category filter request in Burp Suite. I confirmed injection was possible, then used a UNION SELECT payload to check how many columns the query returns and which ones display as text. From there I queried information_schema.tables to find table names, then information_schema.columns to find the username and password columns in the matching users table. Finally, I selected those two columns directly.

Payload (final one):

Gifts' UNION SELECT username_llqgkc, password_ffvkfb FROM users_qiuvzs--

Why it works: the ' closes the original query’s string, UNION SELECT appends a second query returning the data I want, and -- comments out the rest of the original query so it doesn’t cause a syntax error.

Impact: An attacker could dump the entire users table, including the administrator’s password, and fully take over the account — as I did by logging in as administrator.

How to fix it: Use parameterized queries (prepared statements) instead of building SQL with string concatenation, so user input is never treated as executable SQL.
