Lab name: Method-based access control can be circumvented
Category: Access Control
Difficulty: Practitioner

The vulnerability: The admin functionality at /admin-roles is only supposed to be accessible to administrators. The app checks the user’s role, but that check is applied inconsistently depending on the HTTP method used to make the request. A GET request to the endpoint is correctly blocked, but a POST request to the same endpoint skips the access control check entirely.

How I found it: I tried accessing /admin-roles directly as a low-privileged user with a GET request and got "Unauthorized", confirming access control was in place for GET. I then sent the same request as a POST using Burp Repeater. This time I got "Missing parameter 'username'" instead of an unauthorized error — meaning the access control check had been skipped, and the request was now being processed, just missing a required parameter.

I added the missing parameters and successfully promoted another user (carlos) to admin, which also granted my own session visible access to the /admin panel.

Payload (the working request):

POST /admin-roles HTTP/2
Host: [lab-host]
Cookie: session=...
Content-Type: application/x-www-form-urlencoded

username=carlos&action=upgrade

Why it works: the backend’s access control logic likely only runs on the GET handler for this route, or checks the method name explicitly and misses POST. Since the actual privilege-change logic doesn’t re-verify the caller’s role once the method-specific check is bypassed, any authenticated user can perform admin actions just by switching the HTTP method.

Impact: A low-privileged user can escalate their own privileges, or any other account’s privileges, to administrator — a complete bypass of the application’s access control model, leading to full account and data compromise.

How to fix it: Enforce access control centrally (e.g., in middleware or a shared authorization layer) that applies to every HTTP method on a sensitive route, rather than checking permissions separately per method. Never assume an endpoint is “safe” for one HTTP verb just because another verb is protected.


<img width="1918" height="1080" alt="Screenshot From 2026-10-14 20-10-26" src="https://github.com/user-attachments/assets/d4e40f71-929b-45e4-8cd9-ec31988db577" />

<img width="1918" height="1080" alt="Screenshot From 2026-10-14 20-19-13" src="https://github.com/user-attachments/assets/296a406f-9607-4431-9cda-6d348f1a6feb" />

<img width="1918" height="1080" alt="Screenshot From 2026-10-14 20-19-21" src="https://github.com/user-attachments/assets/af97f683-d5cb-45e1-8a65-8465fce77350" />

<img width="1918" height="1080" alt="Screenshot From 2026-10-14 20-13-36" src="https://github.com/user-attachments/assets/89a2904b-177d-4750-99d9-0e96014d7040" />

<img width="1918" height="1080" alt="Screenshot From 2026-10-14 20-15-08" src="https://github.com/user-attachments/assets/ee722f46-8773-42d0-ab03-c070a1ce8759" />

<img width="1918" height="1080" alt="Screenshot From 2026-10-14 20-17-57" src="https://github.com/user-attachments/assets/3b2eb812-eddd-4f78-b6de-5b734ad52bbd" />






