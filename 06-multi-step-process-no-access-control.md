Lab name: Multi-step process with no access control on one step
Category: Access Control
Difficulty: Practitioner

The vulnerability: Promoting a user to admin is a two-step process: an initial request to /admin-roles that shows a confirmation screen, then a second request that actually performs the upgrade. The access control check (verifying the current user is an admin) is only enforced on the first step. The second step, which does the actual privilege change, doesn’t re-check permissions, assuming that if you reached it, you must have already passed the first check.

How I found it: I logged in as the low-privileged user wiener and used Burp to intercept the admin role request flow. I confirmed that step 1 was properly blocked for non-admins. But when I looked at step 2, the confirmation request, it was a simple POST with action=upgrade&confirmed=true&username=. Nothing about that request referenced or depended on completing step 1 first. So I sent step 2 directly, skipping step 1 entirely, with my own username in the parameter.

Payload (the request that bypassed the check):

POST /admin-roles HTTP/2
Host: [lab-host]
Cookie: session=...
Content-Type: application/x-www-form-urlencoded

action=upgrade&confirmed=true&username=wiener

Why it works: the backend trusts that reaching the confirmation step means authorization was already verified, but it never actually checks the user’s role at that point. Since HTTP requests are stateless and each step is really just a separate endpoint, there’s nothing stopping an attacker from calling the final step directly.

Impact: Any authenticated low-privileged user can grant themselves (or anyone) administrator rights by sending a single crafted request, completely bypassing the intended multi-step confirmation flow and its access control.

How to fix it: Apply access control checks at every step of a multi-step process, not just the first. Treat each endpoint as independently reachable and assume an attacker can call any step directly, in any order.


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-13-38" src="https://github.com/user-attachments/assets/81d9b16d-f03b-45de-923e-2ecc773f47e5" />


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-14-15" src="https://github.com/user-attachments/assets/e4753250-a535-4119-beea-192200e6d179" />


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-14-31" src="https://github.com/user-attachments/assets/0eba8a7a-dfbb-40b2-a82b-ac6c9a004ca9" />


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-15-05" src="https://github.com/user-attachments/assets/257ef592-9f43-4c88-b02e-e55d68ad7586" />


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-15-18" src="https://github.com/user-attachments/assets/05bcce91-0edf-433b-b480-1ba6e51acc4f" />


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-17-11" src="https://github.com/user-attachments/assets/0e1aacbe-4247-4a24-8bca-4e71ca6df176" />


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-17-30" src="https://github.com/user-attachments/assets/8a7f74ff-cf1c-41d6-8e9c-44112e70c23a" />


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-20-49" src="https://github.com/user-attachments/assets/22d45251-848d-48c9-a069-f7c0c33cb7fb" />


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-21-21" src="https://github.com/user-attachments/assets/0cf5aeda-2ffe-44bb-ae13-57905b13e250" />


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-21-36" src="https://github.com/user-attachments/assets/19383eee-59dd-4d0d-bb0e-0d6ea178aa2f" />


<img width="1920" height="1080" alt="Screenshot From 2026-10-15 00-21-43" src="https://github.com/user-attachments/assets/60ec3620-17e0-4f42-9993-ca56b46161ac" />









