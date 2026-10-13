Lab name: Blind SQL injection with conditional responses
Category: SQL Injection (Blind)
Difficulty: Practitioner

The vulnerability: The TrackingId cookie is used in a SQL query on the backend, but unlike the earlier lab, the response never shows database output directly. Instead, the application behaves differently depending on whether the injected condition is true or false — in this case, whether the page shows “Welcome back!” or not. That difference is the only signal an attacker gets, which is what makes it “blind.”

How I found it: I intercepted a request in Burp and modified the TrackingId cookie to ' AND '1'='1 (a true condition) and separately to ' AND '1'='2 (a false condition). The true condition kept me logged in and showed “Welcome back!”; the false one didn’t. That confirmed the cookie value was being used directly in a SQL query, and that I could use true/false conditions to ask the database yes/no questions.

From there I built up the attack step by step:

Confirmed a subquery worked: ' AND (SELECT 'a' FROM users LIMIT 1)='a
Narrowed to a specific user: ' AND (SELECT 'a' FROM users WHERE username='administrator')='a
Found the password length using LENGTH(password)>N, increasing N until the condition flipped from true to false (landed on 20 characters).
Used Burp Intruder with a cluster bomb attack to automate pulling out each character with SUBSTRING(password,N,1)='X', cycling through position and character until the response length changed, which signaled a correct guess.

Payload (core technique):

TrackingId=...' AND (SELECT SUBSTRING(password,§1§,1))='§a§

Why it works: the first §1§ is the character position in the password, the second §a§ is the guessed character. Burp tries every combination and the response length tells you which guess was correct, letting you reconstruct the password one character at a time.

Impact: Even with zero visible database output, an attacker can fully extract sensitive data — here, the administrator’s password — purely from true/false behavior differences. This shows that blind SQLi is just as dangerous as regular SQLi, only slower.

How to fix it: Same root fix as regular SQLi — use parameterized queries so user input can never alter the query’s logic, blind or otherwise.
