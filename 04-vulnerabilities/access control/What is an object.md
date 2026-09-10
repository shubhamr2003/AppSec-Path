In an **IDOR (Insecure Direct Object Reference)**, an **object** simply means **a specific piece of data or resource that the application stores or manages**.

PortSwigger describes objects as things such as database records or static files that the application accesses using a user-controlled reference. ([PortSwigger](https://portswigger.net/web-security/access-control/idor?utm_source=chatgpt.com "Insecure direct object references (IDOR) | Web Security Academy"))

### Think of it like this

Suppose you're using an e-commerce website.

Your account has:

- User profile
- Orders
- Invoices
- Messages
- Uploaded documents

Each of these can be an **object**.

For example:

```http
GET /api/orders/1001
```

Here:

- `order` = **object type**
- `1001` = **reference/identifier**
- The actual order belonging to someone = **object**

So conceptually:

```text
Order ID 1001
     ↓
[ Order #1001 ]
     ↓
User: Shubham
Product: Laptop
Amount: ₹50,000
```

If you change:

```http
GET /api/orders/1001
```

to:

```http
GET /api/orders/1002
```

and the server gives you another user's order, you have an **IDOR**.

The problem isn't simply that `1002` exists. The problem is that the server **failed to check whether you are authorized to access object 1002**. ([OWASP Foundation](https://owasp.org/www-community/attacks/insecure_direct_object_reference?utm_source=chatgpt.com "Insecure Direct Object Reference (IDOR) | OWASP Foundation"))

### Objects can be many different things

|Object|Example reference|
|---|---|
|User account|`user_id=123`|
|Order|`order_id=5001`|
|Invoice|`/invoice/1042`|
|Message|`message_id=781`|
|Document|`/documents/123.pdf`|
|Photo|`photo_id=456`|
|Support ticket|`ticket_id=9001`|
|Bank transaction|`transaction_id=7788`|
|API resource|`/api/users/123/profile`|

Even a **filename** can be an object reference:

```http
GET /files/invoice_123.pdf
```

If you change it to:

```http
GET /files/invoice_124.pdf
```

and receive another user's invoice without authorization, that's also an IDOR scenario. ([OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html?utm_source=chatgpt.com "Insecure Direct Object Reference Prevention - OWASP Cheat Sheet Series"))

### The easiest way to remember it

Think:

> **Object = the thing I'm trying to access.**  
> **Reference = how the application identifies that thing.**

For example:

```text
Object:       User's invoice
Reference:    invoice_id=1042
Authorization: Does this user own invoice 1042?
```

A vulnerable application effectively does:

```text
User requests invoice 1042
          ↓
"Does invoice 1042 exist?"
          ↓
YES → Give it to them ❌
```

A secure application does:

```text
User requests invoice 1042
          ↓
"Does invoice 1042 exist?"
          ↓
"Is this user allowed to access it?"
          ↓
YES → Give it to them
NO  → Deny access
```

**That's the key concept behind IDOR.**

And this is why **IDOR is closely connected to horizontal privilege escalation**: if User A changes the reference for their object to User B's object and gets access, User A has moved sideways into another user's resources. ([PortSwigger](https://portswigger.net/web-security/access-control?utm_source=chatgpt.com "Access control vulnerabilities and privilege escalation | Web Security Academy"))

If you're learning this for bug bounty, the next important concept is **how to identify objects/references in a Burp request**—that's where IDOR testing becomes practical.