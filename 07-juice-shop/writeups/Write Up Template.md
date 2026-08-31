# 🔬 [Lab Name]

> **Platform:** PortSwigger Web Security Academy  
> **Category:** XSS  
> **Difficulty:** Apprentice  
> **Status:** ✅ Completed  
> **Date:** YYYY-MM-DD

---

## 🎯 Objective

Briefly describe what the lab requires you to achieve.

---

## 🧠 What I Know Before Starting

What did I already know about this vulnerability?

What was my initial understanding?

---

## 🔎 Recon / Application Analysis

What did I observe while exploring the application?

- Interesting endpoints
    
- Parameters
    
- Forms
    
- Cookies
    
- Headers
    
- User input
    
- Application behavior
    

---

## 💭 Initial Hypothesis

What did I suspect might be vulnerable?

> Example: The search parameter appears to be reflected in the application's response, so I wanted to determine whether the input was being interpreted as HTML.

---

## 🧪 Testing Process

### Step 1 — Identify the Input

Describe where you found the input.

### Step 2 — Send a Test Input

Describe what you tested and why.

### Step 3 — Observe the Response

What changed?

What did the server/application return?

### Step 4 — Confirm the Vulnerability

Explain how you confirmed your hypothesis.

---

## 🛠️ Burp Suite

### Request

```http
GET /search?query=test HTTP/1.1
Host: example.com
```

### Response

Include only the relevant portion.

```http
HTTP/1.1 200 OK
...
```

---

## 💥 Exploitation / Validation

Describe how the vulnerability was demonstrated in the authorized lab environment.

Explain **why** the technique worked rather than simply recording the final payload.

---

## 🎯 Impact

What could this vulnerability allow an attacker to do?

- Account compromise
    
- Sensitive information disclosure
    
- Unauthorized actions
    
- Session compromise
    
- etc.
    

---

## 🛡️ Remediation

How should the vulnerability be prevented?

Mention relevant:

- Input handling
    
- Output encoding
    
- Security controls
    
- Framework protections
    
- Configuration changes
    

---

## ❌ What I Got Wrong

What did I try that didn't work?

Why didn't it work?

---

## 💡 Key Takeaways

---

## 📚 Resources

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
    

---

## 🔗 Related Notes

- [[XSS]]
    
- [[Burp Suite - Repeater]]
    
- [[HTTP Headers]]
    

---

## ✅ Completion

**Lab completed:** YYYY-MM-DD

**Difficulty:** ⭐⭐⭐☆☆

**Confidence:** ⭐⭐⭐⭐☆