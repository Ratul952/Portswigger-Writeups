Lab name: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped
Category: Cross-Site Scripting (Reflected)
Difficulty: Practitioner

The vulnerability: The search parameter is reflected directly inside a JavaScript string (var searchTerms = '...') on the page. The app tries to defend against XSS by HTML-encoding angle brackets and double quotes, and by escaping single quotes with a backslash (\'). But it doesn’t escape the backslash character itself, so an attacker can neutralize the app’s own escaping.

How I found it: I searched for a random string and used Burp to find where it was reflected. It showed up inside a <script> block, inside a JavaScript string literal. I tested a single quote ' and saw the app escaped it to \', which looked safe at first. But since the backslash itself wasn’t escaped, I could send my own backslash right before the app’s quote, so the app’s escaping backslash ends up escaping my backslash instead of my quote — leaving my quote free to close the string.

Payload:

\'-alert(1)//

Why it works: my \ pairs up with the app’s auto-inserted \ in front of the real quote, canceling it out. That leaves my own ' free to close the JavaScript string early. -alert(1) then executes as code, and // comments out the rest of the original line so there’s no syntax error.

Impact: An attacker could run arbitrary JavaScript in a victim’s browser by sending them a crafted link, enabling session hijacking, credential theft, or any action the victim could perform on the site.

How to fix it: Avoid reflecting user input directly into JavaScript string literals. Instead, pass data from the server into JavaScript via safe methods like JSON.stringify() in a context that’s HTML-escaped, or use textContent/dataset attributes instead of inline <script> blocks. Context-aware output encoding (encoding differently depending on whether the value lands in HTML, JS, or a URL) is essential.

<img width="1920" height="1080" alt="1" src="https://github.com/user-attachments/assets/921d275d-7867-4197-bca4-ebe7d7156598" />


<img width="1920" height="1080" alt="2" src="https://github.com/user-attachments/assets/94a1411f-76ac-41ca-b90e-464f874c3405" />


<img width="1920" height="1080" alt="3" src="https://github.com/user-attachments/assets/d0aaee0e-b255-4aca-8e45-842816232619" />


<img width="1920" height="1080" alt="4" src="https://github.com/user-attachments/assets/62eb3b30-d68a-44a8-b170-8944bd543816" />


<img width="1920" height="1080" alt="5" src="https://github.com/user-attachments/assets/d7f52108-44c4-4332-9a33-b3ea1ac56755" />


<img width="1920" height="1080" alt="6" src="https://github.com/user-attachments/assets/c1521662-44d2-4af4-b481-45ca97f7b956" />


<img width="1920" height="1080" alt="7" src="https://github.com/user-attachments/assets/50b8b2f0-9437-4837-8500-9c83b9c60382" />


<img width="1920" height="1080" alt="8" src="https://github.com/user-attachments/assets/782bb6f2-871a-4399-885d-ce60bcf3af09" />







