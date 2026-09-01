resources:

youtube:

## What is IDOR?

**IDOR** stands for **Insecure Direct Object Reference**. It is an access-control vulnerability that occurs when an application uses a user-controlled value—such as an ID, filename, or account number—to access an object without checking whether that user is allowed to access it.[[portswigger](https://portswigger.net/web-security/access-control/idor)][[owasp](https://owasp.org/www-community/attacks/insecure_direct_object_reference)]

An “object” can be:

- A user profile.
- An order.
- A support ticket.
- A document.
- A message.
- A bank statement.
- A product review.
- A database record.
- A file.

**Types of IDOR**
Forced Browsing Error

### Simple example

Suppose a user is allowed to view their own order:

```
GET /api/orders/1001
```

The application should verify that the logged-in user owns order `1001`.

If the user changes the request to:

```
GET /api/orders/1002
```

and receives another customer’s order, the application has an IDOR vulnerability.

The problem is **not simply that the number is visible**. The real problem is that the server trusts the supplied number without checking authorization.

## Authentication versus authorization

This distinction is very important:

- **Authentication:** “Who are you?”
- **Authorization:** “What are you allowed to access or do?”

In an IDOR case, the attacker may be properly authenticated. They may be logged in as an ordinary user, but the server fails to check whether they are authorized to access the specific object.

```
Logged in?                 Yes
Allowed to access object?  No
Application checks this?   No
Result:                    IDOR
```

## Common IDOR forms

### 1. Read IDOR

The attacker can view another user’s information.

```
GET /profile/101
GET /profile/102
GET /profile/103
```

Possible impact:

- Personal information disclosure.
- Private messages exposed.
- Other users’ invoices accessed.
- Customer addresses or phone numbers revealed.
- Internal documents downloaded.

### 2. Update IDOR

The attacker can modify another user’s object.

```
PUT /api/profile/102
```

Possible impact:

- Change another user’s email address.
- Modify an order.
- Edit another person’s review.
- Change delivery or billing information.
- Alter account settings.

### 3. Delete IDOR

The attacker can delete an object belonging to another user.

```
DELETE /api/tickets/102
```

Possible impact:

- Delete another user’s ticket.
- Remove reviews or posts.
- Delete files.
- Cancel orders.
- Destroy records.

### 4. File-based IDOR

The application exposes a predictable filename or path:

```
/download/invoice-1001.pdf
/download/invoice-1002.pdf
```

If changing the filename allows access to another customer’s invoice, this is an IDOR-style access-control failure.

### 5. API IDOR or BOLA

In APIs, IDOR is often called **BOLA**, meaning **Broken Object Level Authorization**. It occurs when an API allows a user to manipulate an object identifier and access another object without proper authorization.[[owasp](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)]

Example:

```
GET /api/users/45/orders
```

The API should verify that the current user is allowed to view the orders of user `45`.

## What can IDOR lead to?

The impact depends on what the affected object represents and which action is possible.

|Impact|Example|
|---|---|
|**Information disclosure**|View another user’s profile, invoice, or private document.|
|**Privacy violation**|Access personal, medical, financial, or contact information.|
|**Unauthorized modification**|Change another user’s email, order, or profile.|
|**Unauthorized deletion**|Delete another user’s post, file, or ticket.|
|**Financial fraud**|Change an order, refund, payment, or wallet object.|
|**Account takeover**|Modify an email address, recovery setting, or password-reset object.|
|**Privilege escalation**|Access administrator-owned resources or restricted functions.|
|**Business disruption**|Cancel orders, remove records, or change important settings.|
|**Data harvesting**|Enumerate many object IDs and collect records in bulk.|
|**Regulatory impact**|Exposure of personal or sensitive data may create compliance problems.|

IDOR does not automatically mean account takeover. For example, viewing another user’s public-looking profile may have limited impact, while modifying a password-reset token or account email could be much more serious.

## Vulnerable and secure logic

### Vulnerable example

A vulnerable application may do something like:

```
order_id = request.args["order_id"]
order = database.get_order(order_id)
return order
```

It retrieves the order based only on the ID supplied by the client. It does not check ownership.

### Safer example

```
order_id = request.args["order_id"]
current_user_id = session["user_id"]

order = database.get_order(
    order_id=order_id,
    owner_id=current_user_id
)

if order is None:
    return "Not found", 404

return order
```

The query is scoped to both:

- The requested object ID.
- The identity or permissions of the logged-in user.

A secure application should perform this check for **every request**, including read, update, and delete operations. OWASP recommends checking whether the authenticated user is authorized to access the specific object, not merely whether the user is logged in.[[owasp](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)][[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/IDOR)]

## How to test IDOR safely

Practise only on a local lab, your own application, or a bug-bounty target where this testing is explicitly permitted.

A safe lab workflow is:

1. Create two test accounts: Account A and Account B.
2. Log in as Account A.
3. Capture a request for an object owned by Account A.
4. Change only the object identifier to an object belonging to Account B.
5. Observe whether the server returns, modifies, or deletes Account B’s object.
6. Stop after confirming the behavior; do not access unnecessary data.
7. Record the request, response, impact, and remediation.

Example comparison:

```
GET /api/orders/1001
```

Change only:

```
GET /api/orders/1002
```

A secure response could be:

```
HTTP/1.1 403 Forbidden
```

or a carefully designed:

```
HTTP/1.1 404 Not Found
```

The important point is that the server must not reveal or modify the unauthorized object.

Use two accounts because comparing requests from different users helps you identify horizontal access-control problems.

## Mitigation

### 1. Perform server-side authorization checks

The server must verify object access on every request:

```
Can this current user perform this action on this specific object?
```

Do not rely on:

- Hidden form fields.
- JavaScript checks.
- Disabled buttons.
- URL secrecy.
- Client-side role values.
- The user simply being logged in.

### 2. Scope database queries to the current user

Instead of:

```
SELECT * FROM orders WHERE id = :order_id;
```

use logic equivalent to:

```
SELECT * FROM orders
WHERE id = :order_id
AND owner_id = :current_user_id;
```

For role-based applications, apply the appropriate organization, tenant, ownership, and role checks.

### 3. Check authorization for every action

Authorization must apply to:

- `GET`.
- `POST`.
- `PUT`.
- `PATCH`.
- `DELETE`.
- File downloads.
- Background jobs.
- GraphQL resolvers.
- WebSocket messages.
- Internal service calls.

Do not secure the “view” endpoint while leaving the “edit” or “delete” endpoint unprotected.

### 4. Use unpredictable identifiers as defense in depth

Instead of sequential values:

```
/orders/1001
/orders/1002
/orders/1003
```

an application might use random identifiers:

```
/orders/550e8400-e29b-41d4-a716-446655440000
```

Random identifiers make enumeration harder, but they are **not a replacement for authorization**. If an attacker obtains a random ID, the server must still check permission. OWASP recommends complex identifiers only as an additional defense.[[owasp](https://owasp.org/www-community/attacks/insecure_direct_object_reference)][[cheatsheetseries.owasp](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)]

### 5. Apply least privilege

Users should have only the access they need.

For example:

- A customer can view their own orders.
- A support employee can view assigned tickets.
- An administrator can access approved management functions.
- One tenant cannot access another tenant’s records.

### 6. Avoid exposing unnecessary properties

Even if a user is allowed to access an object, they may not be allowed to see every field.

For example, a user may be allowed to view an order but not:

- Internal payment references.
- Administrative notes.
- Other users’ information.
- Internal risk scores.

OWASP treats this as a related API authorization problem called **Broken Object Property Level Authorization**.[[owasp](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)]

### 7. Centralize authorization

Put authorization logic in a reusable policy or service instead of implementing slightly different checks in every controller.

This reduces the chance that one forgotten endpoint exposes data.

### 8. Add automated authorization tests

Test combinations such as:

```
User A reading User A's object      → Allowed
User A reading User B's object      → Denied
User A editing User B's object      → Denied
User A deleting User B's object     → Denied
Administrator reading permitted data → Allowed
```

OWASP recommends testing the authorization mechanism and preventing deployment when those tests fail.[[owasp](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)]

### 9. Log suspicious enumeration

Monitor patterns such as:

```
/api/orders/1001
/api/orders/1002
/api/orders/1003
/api/orders/1004
```

Sequential requests for many objects may indicate enumeration. Logging and monitoring can help detect this behavior.[[owasp](https://owasp.org/www-community/attacks/insecure_direct_object_reference)]

## Important takeaway

> **IDOR occurs when a user can change a reference to an object and access or manipulate an object that they are not authorized to use.**

Remember:

```
Visible ID alone ≠ IDOR
Predictable ID alone ≠ IDOR
No server-side object authorization = IDOR risk
```

The strongest mitigation is always:

> **Check authorization on the server for the specific user, object, and action on every request.**


