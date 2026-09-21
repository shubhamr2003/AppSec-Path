resources:
- [intro to dom (medium)](https://medium.com/@anasansari157/introduction-to-dom-6a84befb2884)
- 

yt:
- [DOM Basics](https://www.youtube.com/watch?v=NO5kUNxGIu0)

In JavaScript, **DOM** stands for **Document Object Model**. It’s the browser’s **structured, in-memory representation of the HTML page**, exposed as JavaScript objects that you can read and modify.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

When people say “DOM in JS”, they mean:

> “How JavaScript sees and interacts with the page’s HTML structure, content, and attributes.”

---

## 1) What the DOM actually is

When the browser loads a page like:
```
<!DOCTYPE html>
<html>
  <head>
    <title>My Page</title>
  </head>
  <body>
    <h1 id="title">Hello</h1>
    <p class="text">Welcome</p>
  </body>
</html>
```

the browser builds a **tree of objects** in memory:

- `document` (the root)
    
    - `html`
        
        - `head`
            
            - `title` → text node `"My Page"`
        - `body`
            
            - `h1#title` → text node `"Hello"`
            - `p.text` → text node `"Welcome"`

Each element, attribute, and piece of text becomes a **node** in this tree. JavaScript can:
- Traverse the tree (go from parent to child, sibling to sibling).
- Read properties (text, attributes, classes).
- Modify content, styles, structure.
- Add/remove elements dynamically.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

This tree is the **DOM**.

## 2) How JavaScript accesses the DOM

The global object **`document`** is your main entry point.
Common operations:
### Selecting elements
```
const title = document.getElementById('title');
const paragraphs = document.getElementsByClassName('text');
const firstP = document.querySelector('p');
const allPs = document.querySelectorAll('p');
```

- `getElementById`, `getElementsByClassName`, `getElementsByTagName` → older APIs.
- `querySelector`, `querySelectorAll` → CSS-selector based, more flexible.[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]

### Reading content
```
const titleText = title.textContent;        // "Hello"
const titleHtml = title.innerHTML;          // "Hello" (same here, no nested HTML)
const pText = firstP.textContent;           // "Welcome"
```

- `textContent` → plain text.
- `innerHTML` → HTML as a string (can include tags).[[arxiv](https://arxiv.org/html/2605.25865v1)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

### Modifying content
```
title.textContent = 'Hi there';
firstP.innerHTML = '<strong>Welcome</strong>';
```
Changing `innerHTML` or `textContent` updates what the user sees immediately.

### Creating and adding elements
```
const newP = document.createElement('p');
newP.textContent = 'New paragraph';
document.body.appendChild(newP);
```
This adds a new `<p>` at the end of `<body>`.
### Changing attributes and classes
```
title.setAttribute('data-role', 'main-heading');
title.classList.add('highlight');
title.classList.remove('old-class');
```

## 3) The DOM as a tree of nodes

Key node types:
- **Element nodes**: `<div>`, `<p>`, `<a>`, etc.
- **Text nodes**: the actual text inside elements.
- **Attribute nodes**: attributes like `id`, `class`, `href` (usually accessed via element properties).
- **Document node**: the root `document`.

Basic navigation:
```
const body = document.body;
const firstChild = body.firstChild;
const parent = firstP.parentNode;
const next = firstP.nextSibling;
```

## 4) Events and dynamic behavior

The DOM also defines **events**: user actions or browser events like:
- `click`, `submit`, `input`, `change`
- `load`, `DOMContentLoaded`
- `keydown`, `keyup`, `mouseover`, etc.

You attach handlers:
```
const button = document.querySelector('#submitBtn');

button.addEventListener('click', function () {
  alert('Button clicked!');
});
```
When the user clicks the button, the browser fires a `click` event on that DOM node, and your handler runs.

This is how interactive pages work:  
**HTML structure (DOM) + JavaScript event handlers + dynamic DOM updates = dynamic UI.**[[arxiv](https://arxiv.org/html/2605.25865v1)][[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)]
## 5) The DOM and the browser rendering pipeline

Roughly:
1. Browser downloads HTML.
2. Parses HTML → builds **DOM tree**.
3. Parses CSS → builds **CSSOM** (CSS Object Model).
4. Combines DOM + CSSOM → **render tree**.
5. Layout + paint → pixels on screen.

When you change the DOM with JS:
- The browser may recalculate styles, layout, and repaint.
- Too many rapid changes can hurt performance (reflows/repaints).[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

For security, the key point is:  
**Anything you inject into the DOM (especially via innerHTML, document.write, etc.) can become part of the rendered page and potentially execute scripts.** This is the core of DOM-based XSS.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]
## 6) DOM and security (why it matters for you)
Because the DOM is how JS “sees” the page, many vulnerabilities revolve around it:
### DOM-based XSS

Pattern:
```
const query = location.hash.split('q=')[1];
document.getElementById('results').innerHTML = 'Results for: ' + query;
```
If `query` comes from the URL fragment (`#q=...`) and contains:
```
<img src=x onerror=alert(1)>
```

then `innerHTML` injects it as real HTML → script executes.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]
Here:
- **Source**: `location.hash` (attacker-controlled).
- **Sink**: `innerHTML` (dangerous DOM method).
- No encoding → DOM XSS.
### Other DOM-related issues
- Manipulating `location`, `src`, `href` with untrusted data → open redirects, script inclusion.
- Using `eval()` or `new Function()` with DOM-derived strings → code execution.
- Leaking sensitive data by writing it into the DOM where attacker-controlled scripts can read it.[[acunetix](https://www.acunetix.com/blog/web-security-zone/what-are-insecure-direct-object-references/)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

### Document Object Model
- Object{} that represents the page you see in the web browser and provides you with an API to interact with it. 
- Web browser constructs the DOM when it loads an HTML document, and structures all the elements in a tree-like representation. 
- JavaScript can access the DOM to dynamically change the content, structure, and style of a web page.

The DOM is the browser’s live model of your HTML page. It turns your tags into objects arranged like a tree, so JavaScript can read and change them. 
EX: 
```
<!DOCTYPE html> 
<html> 
	<body> 
		<h1>Hello World</h1> 
		<p>This is a paragraph.</p> 
	</body> 
</html> 
```

DOM tree (in the browser’s memory): 
Document 
	└── html 
		└── body 
			├── h1 │ 
				└── "Hello World" 
			└── p 
				└── "This is a paragraph." 
				
• The DOM is like a tree of folders and files (your page structure). 
• Each tag (<h1>, <p>, <ul>, etc.) is like a folder, and the text/content inside is like a file inside that folder. 
• With JavaScript, you don’t have to open the original HTML file — you just “walk” through the DOM tree (like browsing folders in Explorer) and edit the objects directly. 
• The browser updates the page immediately to match the DOM changes. 
• Meanwhile, your original HTML file on disk stays the same (only the DOM in memory has changed). 

DOM edits are temporary 
• When you use JavaScript to change the DOM (like changing text or colors), it only changes the browser’s live copy of the page. 
• If you refresh the page, those changes are gone, because the browser reloads the original HTML file from disk/server.  

So if you only edit the DOM and never update your actual HTML/CSS files, you’ll “lose” those changes when the page reloads. 
• DOM edits with JavaScript (or DevTools “Inspect” in Chrome) = like a sandbox where you can play, test, and experiment. 
• Refresh = resets everything back to your original HTML/CSS/JS files. 
• If you like the change → you go edit your real HTML/CSS/JS files so the update is permanent. • If you don’t like it → just refresh, and poof it’s gone. 

That’s actually what a lot of developers do daily: 
1. Open DevTools → Elements tab. 
2. Try styling changes, text edits, or even move stuff around (playing with the DOM). 
3. If it looks good, copy those edits back into the real codebase. 

DOM = your test playground 
HTML/CSS files = your real permanent site