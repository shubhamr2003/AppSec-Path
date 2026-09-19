reosurce:

yt:
- [DOM XSS Concept](https://youtu.be/biMtIOR8UAI?si=jgqT6aeO4ApIRL-a)
- 
# DOM Based XSS
DOM-based XSS is a type of cross-site scripting where the **payload never appears in the raw HTML response from the server**; instead, the vulnerable JavaScript running in the browser **reads data from the DOM (or URL) and writes it back into the page unsafely**, allowing your script to execute.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

In other words: the server sends “clean” HTML, but the client-side JS builds part of the page dynamically using untrusted input (like `location.hash`, `location.search`, `document.URL`, or `innerHTML`) without proper encoding, and that’s where the XSS happens.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

---

## How DOM-based XSS differs from reflected/stored XSS

- **Reflected XSS**: malicious input goes to the server and comes back in the HTML response.
- **Stored XSS**: payload is saved on the server (DB, file) and served to victims later.
- **DOM-based XSS**: payload stays entirely on the client side; the server response looks safe, but JS in the page injects it into the DOM unsafely.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

Because of this, scanning the server response alone often misses DOM XSS; you need to analyze the JavaScript and how it uses DOM sources and sinks.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

---

## Core concepts: sources and sinks

### Sources (where untrusted data comes from)

Common DOM/URL sources attackers control:

- `location.href`
- `location.search` (query string)
- `location.hash` (fragment after `#`)
- `document.URL`
- `document.referrer`
- `window.name`
- `localStorage` / `sessionStorage` (if attacker can influence them via another XSS or postMessage)
- `postMessage` data from other origins[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

### Sinks (where data becomes dangerous)

Dangerous DOM “sinks” that can execute script or inject HTML:

- `element.innerHTML = ...`
- `element.outerHTML = ...`
- `document.write(...)`
- `document.writeln(...)`
- `element.insertAdjacentHTML(...)`
- `eval(...)`
- `setTimeout("code string", ...)` / `setInterval("code string", ...)`
- `new Function(...)`
- `location = ...` / `window.location = ...` (for redirects/open-redirect + XSS combos)
- `element.src = ...` when it can load script (e.g., `<script>` tags built dynamically)[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

If untrusted input from a source flows into one of these sinks without proper encoding or validation, you likely have DOM XSS.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

---

## Where to look for DOM-based XSS

When testing a site, focus on:

### 1. Client-side heavy apps

- SPAs (React, Angular, Vue, etc.).
- Pages with lots of JS logic that render content dynamically.
- Routes that use hash-based routing (`#/profile`, `#/search?q=...`).[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

### 2. URL-driven behavior

Look for pages where the URL controls what’s displayed:

- Search pages: `https://example.com/#search?q=term`
- Filter/sort: `https://example.com/#/products?sort=price`
- Redirects: `https://example.com/#redirect?url=...`
- “Return URL”, “next”, “callback” patterns in fragments or query params.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

In Burp:

- Crawl the site, then inspect JS files and HTML for uses of:
    
    - `location.hash`, `location.search`, `document.URL`, etc.
    - `innerHTML`, `document.write`, `eval`, etc.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

### 3. Error messages, search results, dynamic content

Pages that show:

- “No results for: X”
- “Search query: X”
- “Redirecting to: X”
- “Welcome, X” (where X comes from URL or DOM)

are classic candidates if they use unsanitized values in innerHTML or similar.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

---

## How to find DOM XSS in practice

### Step 1: Map inputs and JavaScript behavior

In Burp:

1. Crawl the site (or manually browse key flows).
2. Look at:
    
    - Query parameters (`?q=`, `?search=`, `?next=`, etc.).
    - Hash fragments (`#q=`, `#search=`, `#state=`, etc.).
3. Open the page in a real browser with DevTools → Console / Sources.
4. Search JS for:
    
    - `location.hash`, `location.search`, `document.URL`
    - `innerHTML`, `document.write`, `eval`, `setTimeout("`, `new Function`[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

Browser DevTools “Search” (Ctrl+Shift+F) across all loaded scripts for these patterns.

### Step 2: Inject test payloads and observe the DOM

Use simple payloads to test reflection into dangerous sinks:

```
"><img src=x onerror=alert(1)>
{{constructor.constructor('alert(1)')()}}
<img src=x onerror=console.log('XSS')>
```

Steps:

1. Put your payload in:
    
    - Query params: `?q=<payload>`
    - Hash: `#q=<payload>`
2. Load the page in a real browser (not just Burp Repeater).
3. In DevTools:
    
    - Check the **Elements** tab: does your payload appear as actual HTML?
    - Check the **Console**: does `alert` or `console.log` fire?[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

If your payload executes, you have XSS. If it only appears as text (escaped), it’s likely safe.

### Step 3: Confirm it’s DOM-based

To confirm DOM-based vs reflected:

- View the raw HTML response in Burp (Response tab).
- Search for your payload string.
    
    - If it’s **not** in the server response but still executes in the browser, it’s DOM-based.
    - If it is in the response, it may be reflected/stored instead.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]

---

## Example DOM XSS patterns

### Example 1: Hash-based search

Vulnerable JS:

```
const query = location.hash.split('q=')[1];
document.getElementById('results').innerHTML = 'No results for: ' + query;
```

URL:

```
https://example.com/#search?q=<img src=x onerror=alert(1)>
```

Here:

- Source: `location.hash`
- Sink: `innerHTML`
- No encoding → DOM XSS.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

### Example 2: Redirect with `document.write`

Vulnerable JS:

```
const url = new URLSearchParams(location.search).get('next');
document.write('<a href="' + url + '">Continue</a>');
```

URL:

```
https://example.com/?next="><img src=x onerror=alert(1)>
```

`document.write` directly injects your string into the page → XSS.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

---

## How to exploit DOM-based XSS in bug bounty

Once you’ve found a working DOM XSS:

1. **Craft a reliable payload**  
    Use something clear but non-destructive, e.g.:
    
    ```
    <img src=x onerror=alert(document.domain)>
    ```
    
    Or for modern contexts:
    
    ```
    <svg/onload=alert(document.domain)>
    ```
    
2. **Build a PoC URL**  
    Create a full URL that triggers the XSS when visited:
    
    ```
    [https://target.com/#search?q=](https://target.com/#search?q=)<svg/onload=alert(document.domain)>
    ```
    
3. **Test in a clean browser profile**  
    Ensure it works without your extensions interfering.
4. **Assess impact**  
    Consider:
    
    - Can this be triggered on pages with auth cookies?
    - Does it run in a logged-in context?
    - Can it be delivered via link (phishing, shareable URLs)?  
        DOM XSS that runs in authenticated pages is usually medium–critical.[[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]
5. **Write the report**  
    Include:
    
    - The vulnerable URL with payload.
    - The JS code snippet (file + line) showing source → sink.
    - Steps to reproduce in a fresh browser.
    - Impact (e.g., session theft, actions as user, etc.).[[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

---

## Tools that help

- **Burp Suite**:
    
    - Scanner (Pro) can detect some DOM XSS.
    - Manual: use Repeater + browser for DOM testing.
- **Browser DevTools**:
    
    - Search all scripts for `location.hash`, `innerHTML`, etc.
    - Use breakpoints on suspicious lines.
- **Static analysis tools**:
    
    - ESLint plugins, Semgrep rules, or specialized DOM XSS scanners.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

---

If you want, I can walk through a concrete, step-by-step DOM XSS hunt on a sample app (showing exact JS patterns to search for and how to turn them into an exploit URL).


> **Reflected XSS involves the server reflecting your input into the HTTP response. DOM XSS happens because the browser's JavaScript takes attacker-controlled data and puts it into the page unsafely.**

# DOM-based XSS

DOM XSS can happen **without the server reflecting your input at all**.
Suppose the server sends this perfectly normal page:
```html
<h1 id="welcome"></h1>

<script>
    const name = location.hash.substring(1);

    document.getElementById("welcome").innerHTML =
        "Welcome " + name;
</script>
```

Now you visit:
```text
https://example.com/#yourname
```

The browser's JavaScript does:
```javascript
location.hash
```

which gives:
```text
#yourname
```

Then it puts the value into:
```javascript
innerHTML
```

So the browser creates:
```html
<h1 id="welcome">Welcome yourname</h1>
```

## Now imagine attacker-controlled input

The attacker changes the URL so that the fragment contains HTML.
The important thing is:
```text
URL
 ↓
location.hash
 ↓
JavaScript
 ↓
innerHTML
 ↓
DOM
```
The **server doesn't have to receive or reflect that fragment**.
The browser itself processes the attacker-controlled data.
That's DOM XSS.

# The key difference

|                                   | Reflected XSS                                     | DOM XSS                |
| --------------------------------- | ------------------------------------------------- | ---------------------- |
| Server involved?                  | Yes, usually                                      | Not necessarily        |
| Input reflected in HTTP response? | Yes                                               | Not necessarily        |
| Vulnerability primarily occurs in | Server-side response generation + browser parsing | Client-side JavaScript |
| Important concept                 | Request → response                                | Source → sink          |
| Example source                    | URL parameter reflected by server                 | `location.hash`        |
| Example sink                      | HTML generated by server                          | `innerHTML`            |
| Can server response look normal?  | Usually not                                       | Yes                    |

# The easiest mental model

### Reflected XSS
Think:
> **"I send something to the server, and the server sends my input back to me unsafely."**

```text
Input
 ↓
Server
 ↓
Response
 ↓
Browser
 ↓
XSS
```

### DOM XSS
Think:
> **"The page's JavaScript grabs something I control and puts it somewhere dangerous."**

```text
Attacker-controlled data
          ↓
       SOURCE
          ↓
   Client-side JavaScript
          ↓
        SINK
          ↓
       DOM/XSS
```

# What are "source" and "sink"?
This is **very important for DOM XSS**.
### Source = where the attacker-controlled data comes from

Examples:
```javascript
location.search
location.hash
location.href
document.referrer
```

For example:
```javascript
const input = location.hash;
```

Here:
```text
location.hash = SOURCE
```

### Sink = where the data ends up dangerously

For example:
```javascript
element.innerHTML = input;
```

Here:
```text
innerHTML = SINK
```

So you might see:
```javascript
const input = location.hash;

document.getElementById("output").innerHTML = input;
```

As a pentester, your brain should immediately recognize:
```text
location.hash
     ↓
   SOURCE
     ↓
    input
     ↓
 innerHTML
     ↓
   SINK
```
**That's a potential DOM XSS flow.**

# Why DOM XSS can be confusing

Suppose you use Burp Suite and send:

```http
GET /page?name=test
```

You inspect the server response and don't see:

```text
test
```

anywhere.

You might think:

> "The application isn't reflecting my input, so there's no XSS."

But the page's JavaScript could be doing something like:

```javascript
const name = new URLSearchParams(location.search).get("name");

document.querySelector("#output").innerHTML = name;
```

Now:

```text
?name=test
     ↓
JavaScript reads it
     ↓
innerHTML
     ↓
DOM
```

The server never needed to reflect `test` into the HTML response.

**That's why DOM XSS requires you to look at client-side JavaScript, not just the HTTP response.**

---

## 7. One-line distinction to remember

For your web pentesting notes, remember this:

> **Reflected XSS:** attacker input → **server response** → browser.

> **Stored XSS:** attacker input → **server/database** → later page → browser.

> **DOM XSS:** attacker input → **client-side JavaScript/DOM** → browser.

And the big DOM XSS skill to develop is:

**`SOURCE → DATA FLOW → SINK`**

Once you're comfortable with that, DOM XSS labs in PortSwigger become much easier to understand.