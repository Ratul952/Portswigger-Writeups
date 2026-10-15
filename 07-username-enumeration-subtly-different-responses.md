Lab name: Username enumeration via subtly different responses
Category: Authentication
Difficulty: Practitioner

The vulnerability: The login form shows a generic “Invalid username or password” message for both invalid usernames and valid usernames with wrong passwords, which looks safe at first. But the application was not perfectly consistent: one of the login attempts’ responses differed subtly from the others, by response length. That subtle difference was enough to tell valid usernames apart from invalid ones, even though the visible error text was identical.

How I found it: I used Burp Intruder with a sniper attack against the username parameter, trying a list of common usernames, and set up a grep-extract rule to capture the exact error message text from each response. While every response displayed the same “Invalid username or password” text, I sorted the results by response length and spotted one entry with a different byte length than the rest, revealing that one specific username received a different underlying response even though it rendered the same visible message. That confirmed the username was valid.

Once I had a valid username, I ran a second Intruder attack, this time against the password parameter using a common password wordlist, until one attempt returned a 302 redirect instead of the usual 200 (indicating a successful login rather than a re-rendered login page with an error).

Payload (final working request):

POST /login HTTP/2
Host: [lab-host]

username=adkit&password=[correct-password-from-wordlist]

Why it works: the app’s login logic likely takes a slightly different code path for valid vs. invalid usernames internally (e.g., checking the password hash only for existing users), even though both paths were made to display the same error text. That internal difference leaked into the response itself (length, timing, or formatting), which is enough for an attacker to automate username discovery, then brute-force the password for a confirmed account.

Impact: An attacker can enumerate valid usernames even when the application tries to hide this, then use that list to run targeted password brute-force or credential-stuffing attacks, significantly increasing the odds of account compromise.

How to fix it: Ensure that every code path for failed login attempts (invalid username, valid username with wrong password, etc.) produces an absolutely identical response — same status code, same response length, same timing, same headers. Adding a consistent artificial delay to every login attempt also helps prevent timing-based enumeration.


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 15-17-27" src="https://github.com/user-attachments/assets/75c7290f-f9f8-4873-ac8d-879db945a677" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 15-18-09" src="https://github.com/user-attachments/assets/f6b9312b-75bc-457a-aa8e-57e509ad548d" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-02-24" src="https://github.com/user-attachments/assets/0618f29d-0161-4cba-a496-855ff33039ff" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-04-02" src="https://github.com/user-attachments/assets/230295e1-b54f-4aff-a71c-244b4cdbb844" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-05-02" src="https://github.com/user-attachments/assets/4ea36644-0210-47ec-af6d-c7c40c9c2b5b" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-07-21" src="https://github.com/user-attachments/assets/67812413-9cd0-4f8d-9500-2683d76f238d" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-07-32" src="https://github.com/user-attachments/assets/b88f235f-e34a-4305-81be-f8252f5fdf1d" />






