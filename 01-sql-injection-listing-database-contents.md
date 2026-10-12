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

First Error

<img width="1920" height="1080" alt="Screenshot From 2026-10-12 19-40-36" src="https://github.com/user-attachments/assets/06e1c0cf-7c41-4202-991d-92ebeaa12881" />

Baseline category page

<img width="1920" height="1080" alt="Screenshot From 2026-10-12 19-46-00" src="https://github.com/user-attachments/assets/34d4312c-41f1-42ec-9622-f5c789095537" />

UNION SELECT confirms 2 text columns

<img width="1920" height="1080" alt="Screenshot From 2026-10-12 19-47-13" src="https://github.com/user-attachments/assets/79e56185-821f-4149-acc1-354a4a5ed3f8" />

Listing database tables

<img width="1920" height="1080" alt="Screenshot From 2026-10-12 19-55-34" src="https://github.com/user-attachments/assets/4b3a02b8-3090-4831-b7fa-61995c0c8794" />

Finding username and password columns

<img width="1920" height="1080" alt="Screenshot From 2026-10-12 20-10-01" src="https://github.com/user-attachments/assets/598900f5-e1dc-4a5e-87a9-725cd805b50c" />

Dumped credentials

<img width="1920" height="1080" alt="Screenshot From 2026-10-12 20-15-04" src="https://github.com/user-attachments/assets/6128d1ad-5927-46fe-84a7-2caa5ba50b11" />

Lab Solved

<img width="1920" height="1080" alt="Screenshot From 2026-10-12 20-15-34" src="https://github.com/user-attachments/assets/a23a776e-037f-4a21-a641-0ae066ad8ea5" />


