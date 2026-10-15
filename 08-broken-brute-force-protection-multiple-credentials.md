Lab name: Broken brute-force protection, multiple credentials per request
Category: Authentication
Difficulty: Expert

The vulnerability: This app has brute-force protection that locks an account after too many failed login requests. However, the lockout counter only tracks the number of HTTP requests sent, not the number of individual password attempts. Because the login endpoint accepts JSON and parses the password field as an array, a single request can carry dozens of password guesses at once, and the backend will check all of them against the account without the request count ever increasing.

How I found it: I intercepted a normal login request in Burp and saw it was sent as JSON: {"username":"carlos","password":"test"}. I tested changing the password value to a JSON array of multiple passwords instead of a single string. The server returned a 500 Internal Server Error for a malformed attempt, but after correcting the array syntax, the application accepted it and checked each password in the array against the account, one by one, as part of processing that single request.

I built a request containing the username set to carlos and the password field set to a full array of 100 common passwords (the wordlist I had prepared earlier). Sent as one single request, the server iterated through all 100 guesses internally, and one of them was correct. The lockout defense never triggered, because from its perspective, only one login request had been made.

Payload (the working request):

json
POST /login HTTP/2
Host: [lab-host]
Content-Type: application/json

{
  "username":"carlos",
  "password":[
    "123456","password","qwerty","12345678", ... , "moscow"
  ]
}

Response:

HTTP/2 302 Found
Location: /my-account?id=carlos
Set-Cookie: session=...

Why it works: the brute-force protection counts requests, not individual credential checks. By batching many password guesses into the password array of a single JSON request, an attacker can try an unlimited number of passwords while only ever making one request, completely bypassing any request-rate-based lockout.

Impact: An attacker can brute-force an account’s password at essentially unlimited speed and volume, since the defense meant to stop this kind of attack never triggers. This defeats the entire purpose of brute-force protection and allows fast, large-scale credential-guessing attacks.

How to fix it: Validate and strictly type incoming request parameters (reject arrays where a single string is expected), and count failed login attempts rather than failed requests when applying lockout or rate-limiting logic. Each credential check within a request should count individually toward any brute-force protection threshold.


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-42-32" src="https://github.com/user-attachments/assets/9fbc9046-1caa-4646-b761-89a792569d20" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-43-48" src="https://github.com/user-attachments/assets/846345c1-8092-4657-86c8-55a67395ae97" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-45-34" src="https://github.com/user-attachments/assets/eca11d4c-4ea3-4f3d-ae2b-853a9ee9cf6e" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-48-57" src="https://github.com/user-attachments/assets/f6759cdd-d533-41c7-8d74-7356329ebbe8" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-49-09" src="https://github.com/user-attachments/assets/11673f15-9393-4bb3-88e5-e178656eed90" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-49-36" src="https://github.com/user-attachments/assets/2ffbe690-307b-476c-9211-f20db34f626f" />

<img width="1920" height="1080" alt="Screenshot From 2026-10-15 17-50-05" src="https://github.com/user-attachments/assets/e4f19c31-a77f-4a2c-ac22-ba085d99b292" />





