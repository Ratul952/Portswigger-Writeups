Lab name: Reflected XSS with event handlers and href attributes blocked
Category: Cross-Site Scripting (Reflected)
Difficulty: Practitioner

The vulnerability: The search parameter is reflected into the page’s HTML. The app blocks common XSS vectors, specifically event handler attributes (like onerror, onload) and the href attribute, which are the two most common ways to trigger JavaScript from injected HTML. However, it doesn’t block SVG-specific elements, which opens up a less obvious attack path.

How I found it: Normal payloads using onerror=alert(1) or href="javascript:alert(1)" were stripped or blocked. Since href itself was filtered but animation-based attribute injection wasn’t covered, I used SVG’s <animate> element to dynamically set an href value to a javascript: URI from inside the SVG markup itself, bypassing the direct href filter.

Payload:

html
<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a></svg>

Why it works: instead of writing href="javascript:alert(1)" directly (which gets blocked), the <animate> tag sets the href attribute’s value dynamically at render time. The filter only checks the literal markup, not what the attribute becomes after the SVG animation applies it, so the malicious javascript: URI slips through. Clicking the rendered “Click me” text then executes the script.

Impact: Despite blocking the obvious attack surfaces (event handlers, direct href injection), the app was still vulnerable to arbitrary JavaScript execution via a crafted link. This shows that denylist-based filtering of specific attributes or keywords is incomplete, since there are many indirect ways to achieve the same result.

How to fix it: Don’t rely on blocking specific attribute names or patterns. Use proper context-aware output encoding for any user input placed into HTML, and apply a strict Content Security Policy (CSP) that blocks inline scripts and javascript: URIs as a defense-in-depth measure.


<img width="1920" height="1080" alt="new 1" src="https://github.com/user-attachments/assets/03d27659-541b-4157-bcb3-24c35c946af1" />

<img width="1920" height="1080" alt="new 2" src="https://github.com/user-attachments/assets/1644fd13-35eb-4b94-b8c1-94fdec1bbfc1" />

