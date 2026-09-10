IDOR (Insecure Direct Object Reference), also called BOLA (Broken Object Level Authorization), is one of the most common and high-paying bug bounty classes because it’s a logic flaw: the app exposes an object identifier (ID) and fails to check whether the current user is actually allowed to access that object.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)][[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)]

Below is a practical, field-tested approach to finding IDORs, plus which app types tend to be most vulnerable and how to spot them, drawing on recent writeups and methodology posts from HackerOne Hactivity, Medium, Bugcrowd, The Hacker Recipes, and others.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)][[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[arxiv](https://arxiv.org/html/2605.25865v1)][[infosecwriteups](https://infosecwriteups.com/finding-an-idor-in-user-profile-api-a-15-000-journey-to-critical-0f05e583c00b)][[unihackers](https://unihackers.com/glossary/idor-bola)]

---

## 1) How to identify and test for IDOR (step-by-step)

### Step 1: Create two accounts (minimum)

You need at least:

- **Account A (attacker)** – the user you’re logged in as while testing.
- **Account B (victim)** – another user whose data you’ll try to access/modify.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)][[unihackers](https://unihackers.com/glossary/idor-bola)]

For richer testing, add:

- A low-privilege user vs admin user (for vertical escalation).
- Multiple normal users to see patterns in IDs.[[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)]

### Step 2: Map the app and collect all object references

Use Burp Suite (Proxy + HTTP history) or browser DevTools → Network tab. While logged in as **Account B**, perform realistic actions:

- View profile, orders, invoices, messages, files, subscriptions.
- Create posts, comments, tickets, carts, addresses, payment methods.
- Download documents, export reports, change settings.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)]

Record every identifier you see in:

- **URL path**: `/api/orders/98432`, `/user/profile/johndoe`
- **Query params**: `?userId=5001`, `?fileId=xyz789`
- **Request body (JSON/form)**: `{"recipientId":"user_abc123"}`
- **Headers** (less common but possible): `X-User-ID`, `X-Account`
- **Encoded/hashed IDs**: base64, hex, MD5-like strings.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)][[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)]

These are your **IDOR targets**.

### Step 3: Replay requests as Account A with Account B’s IDs

Log in as **Account A**. For each captured request from Account B:

- Replace the object ID with **Account B’s ID**.
- Send the request and inspect:
    
    - Status code (200, 403, 404, 500).
    - Response body: does it contain B’s data?
    - Any differences vs when using A’s own ID.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[unihackers](https://unihackers.com/glossary/idor-bola)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]

Test all HTTP methods:

- **GET** → read IDOR (viewing others’ data).
- **PUT/PATCH/POST** → write IDOR (modifying others’ data).
- **DELETE** → delete IDOR (removing others’ data).
- Function-level actions (e.g., `/reply`, `/approve`, `/transfer`).[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)][[unihackers](https://unihackers.com/glossary/idor-bola)]

Also test **unauthenticated** requests where possible:

```
curl https://api.example.com/orders/98432
# no Authorization header
```

If this returns data, that’s often critical.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)]

### Step 4: Look for patterns in IDs

Common patterns that frequently yield IDORs:

- **Numeric sequential IDs**: `/orders/1001` → try `1000`, `999`, `1002`, small integers (1, 2, 3 often admin).[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[reddit](https://www.reddit.com/r/cybersecurity/comments/1stjx22/stuck_in_tutorial_hell_i_know_the_theory_of_idor/)]
- **UUIDs/GUIDs**: not safe by themselves; if exposed in public links, emails, or other endpoints, they can still be abused.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)]
- **Encoded IDs**: base64 (`MTIzNDU2`), hex, hashes. Decode them; if they map to simple integers, test those.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)][[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)]
- **Usernames/emails as IDs**: `?username=johndoe` → try other known usernames.[[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)]

Tools like CyberChef or simple CLI commands help decode:

```
echo -n "MTIzNDU2" | base64 -d
```

### Step 5: Focus on state-changing and sensitive actions

IDORs that only expose low-sensitivity data may be rated lower; those that allow:

- Reading **PII**, messages, payment methods, orders.
- Modifying **profile**, email, password, role, address.
- Deleting data or performing actions on behalf of another user.

are typically **high/critical**.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[infosecwriteups](https://infosecwriteups.com/finding-an-idor-in-user-profile-api-a-15-000-journey-to-critical-0f05e583c00b)][[unihackers](https://unihackers.com/glossary/idor-bola)]

Always test:

- Profile updates, email/phone changes.
- Password reset flows.
- Subscription/payment changes.
- Ownership transfers, permission changes.
- Admin-only endpoints accessed as normal user (vertical IDOR).[[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)]

### Step 6: Automate carefully (after manual validation)

Once you understand the pattern:

- Use **Burp Intruder** to fuzz ID parameters with:
    
    - Your own IDs.
    - Known IDs from other accounts.
    - Numeric ranges if IDs look sequential.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)]
- Use **ffuf** or similar for API fuzzing:

```
ffuf -u "https://api.example.com/api/orders/FUZZ" \
  -w id_list.txt \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -fc 403,404
```

Always respect program rules; many bug bounty programs discourage or forbid heavy automated scanning.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)]

---

## 2) Which types of applications are more vulnerable to IDOR?

Recent empirical work and writeups show IDOR/BOLA is especially common in:

### APIs-first and mobile backends

- REST/GraphQL APIs powering web and mobile apps.
- Endpoints like:
    
    - `/api/users/{id}`
    - `/api/orders/{id}`
    - `/api/documents/{id}/download`
    - GraphQL queries: `user(id: "...") { email, orders { ... } }`[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[arxiv](https://arxiv.org/html/2605.25865v1)][[unihackers](https://unihackers.com/glossary/idor-bola)]

Why:

- Rapid development, many endpoints.
- Developers focus on functionality, not per-object authorization checks.
- Mobile apps often call APIs directly with IDs obtained from previous responses.[[arxiv](https://arxiv.org/html/2605.25865v1)][[brightsec](https://brightsec.com/blog/broken-access-control-attack-examples-and-4-defensive-measures/)]

### Multi-tenant SaaS and collaboration platforms

Examples:

- Project management tools.
- Document sharing / cloud storage.
- Team chat, ticketing, CRM, HR platforms.

Look for:

- `?accountId=`, `?orgId=`, `?workspaceId=`, `?projectId=`.
- Invites, shared links, public/private toggles.
- Cross-tenant access (user from Org A accessing Org B’s data).[[arxiv](https://arxiv.org/html/2605.25865v1)][[unihackers](https://unihackers.com/glossary/idor-bola)]

### E-commerce, fintech, and any app with “orders/invoices/payments”

High-value objects:

- Orders, invoices, receipts.
- Payment methods, payouts, transactions.
- Addresses, KYC documents.

These often have:

- Sequential or predictable IDs.
- Complex workflows (create → update → refund → cancel) where auth checks are inconsistently applied.[[arxiv](https://arxiv.org/html/2605.25865v1)][[infosecwriteups](https://infosecwriteups.com/finding-an-idor-in-user-profile-api-a-15-000-journey-to-critical-0f05e583c00b)][[unihackers](https://unihackers.com/glossary/idor-bola)]

### Social, content, and messaging platforms

Objects:

- Posts, comments, messages, chats.
- Media uploads, stories, reels.
- Notifications, followers/following lists.

IDOR shows up in:

- “View another user’s DMs”.
- Accessing private posts via direct URL.
- Modifying or deleting others’ content.[[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)][[unihackers](https://unihackers.com/glossary/idor-bola)][[reddit](https://www.reddit.com/r/cybersecurity/comments/1stjx22/stuck_in_tutorial_hell_i_know_the_theory_of_idor/)]

### Apps with “shareable links” or public URLs for private resources

If private resources are accessible via a link like:

- `https://app.com/document/abc123`
- `https://app.com/invite/xyz789`

and the same ID is used internally without extra checks, you can often:

- Guess or harvest IDs.
- Access other users’ “private” resources.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[unihackers](https://unihackers.com/glossary/idor-bola)]

---

## 3) How to identify IDOR-prone apps during recon

When scanning a target program, prioritize apps that show these signals:

- **Heavy API usage**:
    
    - Lots of `/api/...` paths in JS files or network traffic.
    - Mobile apps (check Play Store / App Store linked in scope).[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[arxiv](https://arxiv.org/html/2605.25865v1)]
- **User-generated objects**:
    
    - Orders, tickets, posts, files, messages, subscriptions.
    - Dashboards showing lists of “your” resources.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[reddit](https://www.reddit.com/r/cybersecurity/comments/1stjx22/stuck_in_tutorial_hell_i_know_the_theory_of_idor/)]
- **Identifiers in URLs/params**:
    
    - Numeric IDs, UUIDs, base64/hex strings tied to resources.
    - Parameters like `userId`, `orderId`, `fileId`, `accountId`.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)][[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)]
- **Multi-user or multi-tenant features**:
    
    - Workspaces, organizations, teams, projects.
    - Role-based views (admin vs user).[[arxiv](https://arxiv.org/html/2605.25865v1)][[unihackers](https://unihackers.com/glossary/idor-bola)]
- **Recent feature launches / fast iteration**:
    
    - New endpoints, beta features, mobile app updates.
    - These often have incomplete access control.[[arxiv](https://arxiv.org/html/2605.25865v1)]

Reading **HackerOne Hactivity** and **Medium writeups** for a specific company can also reveal:

- Which endpoints have had IDOR/BOLA before.
- Patterns in their ID formats and API structure.[[arxiv](https://arxiv.org/html/2605.25865v1)][[infosecwriteups](https://infosecwriteups.com/finding-an-idor-in-user-profile-api-a-15-000-journey-to-critical-0f05e583c00b)][[infosecwriteups](https://infosecwriteups.com/tips-that-worked-for-me-039da09584c5)]

---

## 4) Example IDOR testing flow (practical checklist)

Use this as a compact checklist while testing:

1. Register **two accounts** (A and B).[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[unihackers](https://unihackers.com/glossary/idor-bola)]
2. As B, perform key actions; log all requests with IDs in Burp.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)]
3. Categorize IDs: numeric, UUID, encoded, usernames, etc.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)][[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)]
4. As A, replay B’s requests, swapping in B’s IDs.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[aikido](https://www.aikido.dev/blog/idor-vulnerability-explained)]
5. Test:
    
    - Read (GET).
    - Update (PUT/PATCH/POST).
    - Delete.
    - Function-level actions (reply, approve, transfer).[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)][[unihackers](https://unihackers.com/glossary/idor-bola)]
6. Try unauthenticated requests to the same endpoints.[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)]
7. Decode any encoded IDs; test underlying values.[[thehacker](https://www.thehacker.recipes/web/inputs/idor)][[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[herish](https://herish.me/blog/how-to-test-for-idor-practical-methodology/)]
8. Assess impact:
    
    - What data is exposed/modified?
    - Can this be automated at scale?
    - Does it affect many users or just one?[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[infosecwriteups](https://infosecwriteups.com/finding-an-idor-in-user-profile-api-a-15-000-journey-to-critical-0f05e583c00b)]

---

## 5) Reporting IDOR effectively (bug bounty angle)

A strong report should clearly state:

- **What** object is accessible (orders, profiles, messages, etc.).
- **How** to reproduce (exact requests, IDs, accounts).
- **Impact** at scale (PII exposure, account takeover, financial risk).[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)][[infosecwriteups](https://infosecwriteups.com/finding-an-idor-in-user-profile-api-a-15-000-journey-to-critical-0f05e583c00b)]

Example summary style:

> An authenticated user can access any other user’s order history by substituting their order ID in `/api/orders/{id}`. No authorization check is performed, allowing mass harvesting of PII (name, address, items, partial payment info).[[dev](https://dev.to/kai_learner/how-to-find-idor-vulnerabilities-the-bug-bounty-hunters-practical-guide-46o8)]

This aligns with how triagers on HackerOne, Bugcrowd, and similar platforms evaluate severity.[[arxiv](https://arxiv.org/html/2605.25865v1)][[infosecwriteups](https://infosecwriteups.com/finding-an-idor-in-user-profile-api-a-15-000-journey-to-critical-0f05e583c00b)]

---

If you want, I can next:

- Walk through a concrete Burp-based IDOR test on a sample API request (with exact Intruder setup).
- Or help you design a small Python script to automate IDOR checks against an API you’re authorized to test.