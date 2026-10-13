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

<img width="1920" height="934" alt="01-baseline" src="https://github.com/user-attachments/assets/aac93b27-7c7d-49a1-8ed8-fd4fccf61788" />


<img width="1920" height="1080" alt="02-true-condition" src="https://github.com/user-attachments/assets/cc3ad458-89e1-444a-b17b-7aa8828ef591" />


<img width="1920" height="1080" alt="03-false-condition" src="https://github.com/user-attachments/assets/e060a5a0-6946-425a-acaa-20a50de54b01" />


<img width="1920" height="1080" alt="04-blind-confirmed" src="https://github.com/user-attachments/assets/2030697b-7404-4a1d-8d69-911e7ea1cda8" />

<img width="1920" height="1080" alt="05-subquery-test" src="https://github.com/user-attachments/assets/cd332d64-728c-461d-ab26-0a7c966e2845" />


<img width="1920" height="1080" alt="06-target-user" src="https://github.com/user-attachments/assets/14979ae6-6f9f-47b1-9fe8-3f6163ce6760" />


<img width="1920" height="1080" alt="07-table-loads" src="https://github.com/user-attachments/assets/4276c6a7-9f70-44e9-98f5-a05382f36b4f" />


<img width="1920" height="1080" alt="08-length-test-1" src="https://github.com/user-attachments/assets/f0067e84-6565-4e3d-9ddb-c14f06943e92" />


<img width="1920" height="1080" alt="09-length-19" src="https://github.com/user-attachments/assets/74050f49-37ac-4fd7-bd2d-5da014b5040d" />


<img width="1920" height="1080" alt="10-length-20" src="https://github.com/user-attachments/assets/5d09c141-3841-4376-9384-3cd4c97eb0ae" />


<img width="1920" height="1080" alt="11-intruder-setup" src="https://github.com/user-attachments/assets/2f07aca4-c59e-4904-8c38-9550389e5f91" />


<img width="1920" height="1080" alt="12-intruder-results" src="https://github.com/user-attachments/assets/90692176-428d-4710-ad94-1f778213e4df" />


<img width="1920" height="1080" alt="14-char-extraction" src="https://github.com/user-attachments/assets/be51e560-2302-41fc-8dcf-22155fde18c0" />

<img width="1920" height="1080" alt="13-solved" src="https://github.com/user-attachments/assets/a05eb543-6438-49fd-965a-066e6420931c" />












