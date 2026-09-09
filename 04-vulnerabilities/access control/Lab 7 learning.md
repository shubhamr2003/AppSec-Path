**Not every horizontal privilege escalation is IDOR**
They are very closely related, but they are not exactly the same thing.

The easiest way to remember it:

> **IDOR is a vulnerability pattern. Horizontal privilege escalation is an impact/result.**

### IDOR

**IDOR = Insecure Direct Object Reference**

It happens when an application exposes an object identifier and doesn't properly check whether you are allowed to access that object.

Example:

```http
GET /api/users/1001/profile
```

You are logged in as user `1001`.

You change it to:

```http
GET /api/users/1002/profile
```

If you can see user `1002`'s private information:

```text
Your account
     ↓
Change ID
     ↓
Another user's account
     ↓
Access granted ❌
```

That's an **IDOR**.

---

### Horizontal privilege escalation

Horizontal privilege escalation means:

> **A user gains access to another user who has roughly the same privilege level.**

For example:

```text
User A
  ↓
accesses
  ↓
User B's account/data
```

Both are normal users:

```text
User A → User
User B → User
```

That's **horizontal** escalation.

---

### How they overlap

A very common scenario is:

```text
IDOR
 ↓
Access another user's resource
 ↓
Horizontal privilege escalation
```

For example:

```http
GET /api/orders/1234
```

Your order:

```text
1234 → Your order
```

Change:

```http
GET /api/orders/1235
```

If `1235` belongs to another normal user and you can access it:

**Vulnerability:** IDOR / broken object-level authorization

**Result:** Horizontal privilege escalation

---

### But IDOR isn't always horizontal

Suppose:

```http
GET /api/users/1001
```

allows you to access an **administrator's** data:

```text
Normal user
     ↓
Admin's resource
     ↓
Access granted
```

That's still potentially an **IDOR**, but the privilege boundary you're crossing is **vertical**.

### For your learning

Since you're currently studying **Broken Access Control**, learn these as separate concepts:

**IDOR/BOLA**  
→ Can I access someone else's object?

**Horizontal privilege escalation**  
→ Can I perform/access things belonging to another user at my privilege level?

**Vertical privilege escalation**  
→ Can I perform/access things reserved for a higher-privileged user?

And remember:

> **IDOR can be the mechanism that causes horizontal or vertical privilege escalation, but they aren't synonyms.**
### When horizontal escalation IS IDOR

Suppose you're User A:

```http
GET /api/profile/1001
```

Your ID is `1001`.

You change it:

```http
GET /api/profile/1002
```

and see User B's profile.

Here:

- **IDOR/BOLA** → you accessed another user's object by manipulating its identifier.
    
- **Horizontal privilege escalation** → you crossed from User A → User B.
    

So **one vulnerability can be both**.

---

### Horizontal escalation WITHOUT IDOR

Imagine the application has:

```http
POST /api/user/change-email
```

There is no user ID in the request.

Your account is:

```text
User A
```

But because of a broken authorization check, you discover that you can call an endpoint/function that changes **another user's email** based on some server-side state or another non-ID mechanism.

That's still:

**Horizontal privilege escalation**

but it isn't necessarily an IDOR.

---

### Easy way to remember

Ask **two different questions**:

**Question 1 — What am I accessing?**

> Am I accessing another user's object by manipulating an identifier?

➡️ **IDOR/BOLA**

**Question 2 — What privilege boundary did I cross?**

> Did I go from my account to another user with similar privileges?

➡️ **Horizontal privilege escalation**

So you can have:

```text
IDOR + Horizontal escalation
        ↓
User A → User B
```

or:

```text
IDOR + Vertical escalation
        ↓
User → Admin
```

or:

```text
Horizontal escalation
        ↓
User A → User B
        ↓
without an IDOR
```

### For bug bounty reports

Don't automatically label something **IDOR** just because another user's data/action is accessible.

First identify **the actual root cause**.

If you manipulated:

```text
userId=1002
```

or:

```text
/order/1002
```

to access another user's object → **IDOR/BOLA is a strong classification**.

If the issue is simply that authorization isn't enforced for a privileged action, it may be **broken access control / privilege escalation** without being IDOR.