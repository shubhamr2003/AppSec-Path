resources:

yt:
- [types of authentication](https://youtu.be/iX8g4LqF8p8)
- 

![](Pasted%20image%2020260924180154.png)

## Difference between Authentication Types, Authentication Methods and Authorization Frameworks

```text
                 SECURITY
                    │
        ┌───────────┴───────────┐
        │                       │
 AUTHENTICATION             AUTHORIZATION
 "Who are you?"             "What can you do?"
        │                       │
   Types/Methods             Models/Frameworks
```

## 1. Authentication Type

An **authentication type** describes the general way identity is established.

The classic categories are based on the authentication factor:

### Something you know

```text
Password
PIN
Security question
```

### Something you have

```text
Phone
Authenticator app
Hardware security key
Smart card
```

### Something you are

```text
Fingerprint
Face
Iris
```

### Somewhere you are

```text
Location/IP/network
```

### Something you do

```text
Behavioral characteristics
Typing patterns
```

So **password authentication** is based on "something you know."

---

# 2. Authentication Method

A **method** is the specific mechanism/protocol used to authenticate the user.

For example:

|Method|What it does|
|---|---|
|Username + Password|Verifies credentials|
|Session Cookie|Maintains an authenticated session|
|HTTP Basic Auth|Sends credentials using `Authorization`|
|Bearer Token|Authenticates using a token|
|JWT|Token format commonly used for authentication|
|API Key|Identifies/authenticates an API client|
|TOTP|Verifies a time-based OTP|
|Client Certificate|Uses a certificate to authenticate|
|OAuth 2.0|Delegates access; commonly part of login architectures|
|OpenID Connect|Provides authentication/identity on top of OAuth 2.0|
|SAML|Federated authentication/SSO|

For example:

```text
Authentication type:
Something you know

        ↓

Authentication method:
Username + Password

        ↓

Application:
POST /login
username=shubham
password=********
```

---

# 3. Authentication Mechanism

You may also hear **mechanism**.

This usually refers to **how the authentication method is technically implemented**.

For example:

```text
Username + Password
        ↓
Server verifies password
        ↓
Creates session
        ↓
Set-Cookie: session=abc123
```

Here you have:

```text
Factor/type → Something you know
Method      → Username + Password
Mechanism   → Session cookie maintains login
```

The terminology isn't perfectly standardized, though. Different documentation and security teams may use "method," "mechanism," and "scheme" differently.

---

# 4. Authorization

Now we move to the other question:

> **What is this authenticated user allowed to do?**

Example:

```text
Authentication

Who are you?
       ↓
User: Shubham


Authorization

What can Shubham do?
       ↓
Read profile       ✅
Edit profile       ✅
Delete account     ❌
Admin panel        ❌
```

---

# 5. Authorization Model

An **authorization model** defines the rules for deciding what users can access.

### RBAC — Role-Based Access Control

Permissions are assigned to roles.

```text
User
 ↓
Role
 ↓
Permissions
```

Example:

```text
Shubham
   ↓
Employee
   ↓
Read reports
Create tickets
```

Admin:

```text
Admin
 ↓
Read reports
Create tickets
Delete users
Manage settings
```

---

### ABAC — Attribute-Based Access Control

Access depends on attributes.

For example:

```text
User = Shubham
Role = Employee
Department = Security
Location = Mumbai
Time = 10:00
```

The system evaluates these attributes against a policy.

---

### DAC — Discretionary Access Control

The owner of a resource can decide who gets access.

A simple example is a file owner granting another user access to a file.

---

### MAC — Mandatory Access Control

Access is determined by centrally enforced security labels/policies.

Commonly associated with highly controlled environments.

---

# 6. Authorization Framework

A **framework** is generally a standardized specification/ecosystem for implementing authorization or delegated access.

The most important one for your **API/web pentesting learning** is:

## OAuth 2.0

OAuth 2.0 is an **authorization framework**.

Conceptually:

```text
User
 │
 ▼
Application
 │
 ▼
Authorization Server
 │
 │ Access Token
 ▼
API / Resource Server
```

OAuth allows an application to obtain limited access to resources without necessarily receiving the user's password.

Important OAuth concepts:

```text
Client
Resource Owner
Authorization Server
Resource Server
Access Token
Refresh Token
Scope
Redirect URI
```

---

# 7. What about OpenID Connect?

This is another important distinction.

**OAuth 2.0 → authorization/delegated access**

**OpenID Connect (OIDC) → authentication/identity layer built on OAuth 2.0**

For example:

```text
OAuth 2.0
   ↓
"Can this application access this resource?"

OIDC
   ↓
"Who is the user?"
```

That's why saying:

> "OAuth is an authentication protocol"

is usually an oversimplification.

---

# 8. SAML

SAML is commonly used for **federated identity and SSO**.

For example:

```text
Employee
   ↓
Company Identity Provider
   ↓
SAML assertion
   ↓
Application
   ↓
Logged in
```

You'll encounter SAML frequently in enterprise environments.

---

# Put everything together

Here's the hierarchy I recommend you remember:

```text
                    SECURITY
                       │
             ┌─────────┴─────────┐
             │                   │
      AUTHENTICATION        AUTHORIZATION
      "Who are you?"        "What can you do?"
             │                   │
             │                   │
        Authentication      Authorization
        factors/types       models
             │                   │
      ┌──────┼──────┐       ┌────┼────┐
      │      │      │       │    │    │
   Know   Have   Are       RBAC ABAC DAC
      │
      ▼
 Authentication
 methods
      │
 ┌────┼───────────┐
 │    │           │
Password Session  Token
       │           │
       │      ┌────┼─────┐
       │      │    │     │
       │     JWT  API   Bearer
       │          Key   Token
       │
       ▼
 Authentication
 mechanisms
```

And separately:

```text
AUTHORIZATION FRAMEWORKS / STANDARDS
            │
      ┌─────┴─────┐
      │           │
   OAuth 2.0    SAML
      │
      ▼
    OIDC
```

## For your web/API pentesting path

I'd learn them in this order:

```text
1. Authentication vs Authorization
          ↓
2. Password authentication
          ↓
3. Sessions & cookies
          ↓
4. HTTP Basic Authentication
          ↓
5. Bearer tokens
          ↓
6. JWT
          ↓
7. API keys
          ↓
8. MFA / TOTP
          ↓
9. OAuth 2.0
          ↓
10. OpenID Connect
          ↓
11. SAML / SSO
          ↓
12. RBAC / ABAC
          ↓
13. Authorization vulnerabilities
      ├── IDOR/BOLA
      ├── Horizontal privilege escalation
      ├── Vertical privilege escalation
      └── Function-level access control
```

**The key distinction to memorize:**

> **Authentication method** = _How does the application verify your identity?_  
> **Authorization model** = _How does the application decide what you're allowed to access?_  
> **Authorization framework** = _What standardized mechanism/framework can be used to delegate or establish access permissions?_