resources:
https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS

youtube:
- [What is CORS](https://youtu.be/WWnR4xptSRk)
- [Portswigger CORS Lab 1](https://youtu.be/RKe6r63OxXw)
- [Preflight Request](https://youtu.be/tcLW5d0KAYE)
- [Rana Khalil](https://youtu.be/t5FBwq-kudw)
## What Is CORS?

**CORS** stands for **Cross-Origin Resource Sharing**. It is a browser security mechanism that allows a website to request resources from a different **origin** when the destination server explicitly permits it.

It is a **browser feature**.

**CORS itself is not a vulnerability.** CORS (Cross-Origin Resource Sharing) is a **browser security mechanism** that controls which origins are allowed to access resources from another origin.

The vulnerability occurs when **CORS is misconfigured**.

In Same Origin Policy (SOP) the request if being received by the browser but it blocks the response if there's a mismatch in the origin i.e. `Access-Control-Allow-Origin` 

SOP doesn't provide protection against CSRF, as SOP mainly prevents one origin from reading data from another origin. The request is always accepted by the browser but response is not given. In the case of CSRF the attacker doesn't need a response from the browser, he simply sends a payload that can change the users email id.

SOP prevents scripts from reading cross-origin responses, but browsers can still send certain cross-origin requests. CSRF exploits this behavior to perform unauthorized actions using the victim's credentials

For example:
- Frontend: `https://shop.example`
- API: `https://api.example`

These are different origins, but CORS can allow the frontend to communicate with the API.
## What Is an Origin?

ORIGIN = scheme://host:port

```
scheme://host:port
```

Example:

```
https://example.com:443
```

Two URLs have different origins if their scheme, host, or port differs.

| URL                           | Compared with `https://example.com`           |
| ----------------------------- | --------------------------------------------- |
| `https://example.com/profile` | Same origin, different path (path is allowed) |
| `http://example.com`          | Different scheme                              |
| `https://api.example.com`     | Different host                                |
| `https://example.com:8443`    | Different port                                |

Even `https://example.com` and `https://api.example.com` are different origins because their hostnames differ.

## Why Does CORS Exist?

Browsers use a security rule called the **Same-Origin Policy**, or SOP. It prevents JavaScript running on one website from freely reading sensitive data from another website. CORS provides a controlled exception to that rule.[[owasp](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/07-Testing_Cross_Origin_Resource_Sharing)][[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy)]

Imagine you are logged in to your bank:

```
https://bank.example
```

If any website could silently read your bank account data through JavaScript, that would be dangerous. The browser therefore blocks cross-origin reading unless the bank’s server says that the requesting origin is allowed.

CORS does not completely remove the Same-Origin Policy. It allows the server to specify which other origins may access its responses.[[portswigger](https://portswigger.net/web-security/cors)]

## Simple CORS Example

Suppose JavaScript on this website:

```
https://frontend.example
```

requests data from:

```
https://api.example
```

The browser sends an `Origin` header:

```
GET /users HTTP/1.1
Host: api.example
Origin: https://frontend.example
```

If the API trusts that frontend, it responds:

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://frontend.example
Content-Type: application/json
```

The browser sees that the API allows `https://frontend.example`, so it makes the response available to the frontend’s JavaScript.

If the server does not allow that origin, the browser blocks the JavaScript from reading the response.

## Important CORS Headers

| Header                             | Purpose                                                       |
| ---------------------------------- | ------------------------------------------------------------- |
| `Origin`                           | Identifies the website making the cross-origin request.       |
| `Access-Control-Allow-Origin`      | Specifies which origin may read the response.                 |
| `Access-Control-Allow-Methods`     | Lists permitted methods such as `GET`, `POST`, or `DELETE`.   |
| `Access-Control-Allow-Headers`     | Lists non-simple request headers the client may send.         |
| `Access-Control-Allow-Credentials` | Allows credentials such as cookies to be included.            |
| `Access-Control-Max-Age`           | Tells the browser how long it may cache the preflight result. |

Example:

```
Access-Control-Allow-Origin: https://frontend.example
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Authorization, Content-Type
```

This means:

- Only `https://frontend.example` is trusted.
- `GET` and `POST` requests are allowed.
- The frontend may send `Authorization` and `Content-Type` headers.

## What Is a Preflight Request?

For some cross-origin requests, the browser first sends an `OPTIONS` request to ask the server whether the real request is allowed. This is called a **preflight request**.[[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)][[owasp](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/07-Testing_Cross_Origin_Resource_Sharing)]

For example:

```
OPTIONS /users HTTP/1.1
Host: api.example
Origin: https://frontend.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization, Content-Type
```

The server may respond:

```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://frontend.example
Access-Control-Allow-Methods: POST
Access-Control-Allow-Headers: Authorization, Content-Type
```

The browser then sends the actual `POST` request.

A preflight commonly happens when:

- The method is not a simple `GET`, `POST`, or `HEAD`.
- The request uses methods such as `PUT`, `PATCH`, or `DELETE`.
- The request includes custom headers.
- The request uses certain content types, such as JSON.

## CORS and Cookies

By default, browsers do not include cross-origin cookies in many requests. A frontend must explicitly request credentialed access:

```
fetch("https://api.example/profile", {
  credentials: "include"
});
```

The server must then respond with:

```
Access-Control-Allow-Origin: https://frontend.example
Access-Control-Allow-Credentials: true
```

A server should not use this insecure combination:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

For credentialed requests, the server should specify an exact trusted origin rather than allowing every origin. OWASP recommends allowing only selected, trusted domains and avoiding sensitive cross-origin exposure.[[cheatsheetseries.owasp](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html)][[cheatsheetseries.owasp](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)]

## CORS Security Misconfiguration

Common mistakes include:

```
Access-Control-Allow-Origin: *
```

This allows any origin to read responses that are not protected by credentials. It may be acceptable for genuinely public data, but it is dangerous for sensitive information.

Other problems include:

- Reflecting any value from the `Origin` header without validation.
- Trusting attacker-controlled subdomains.
- Allowing credentials for untrusted origins.
- Allowing unnecessary methods such as `DELETE`.
- Allowing sensitive headers unnecessarily.
- Applying permissive CORS rules to the entire application instead of selected endpoints.

As a pentester, test whether an untrusted origin can read sensitive responses, especially responses containing account information, personal data, or API keys. CORS testing should be performed only against systems you own or are explicitly authorized to test.

## Common Misunderstandings

### CORS is not authentication

CORS does not log users in and does not replace access control. The server must still authenticate the user and authorize every requested resource.

### CORS is mainly a browser restriction

CORS controls whether browser-based JavaScript can read a response. Tools such as `curl`, Burp Suite, or server-side programs are not stopped by the browser’s CORS enforcement. The server must still implement proper authentication and authorization.

### CORS does not automatically prevent CSRF

CORS and CSRF are related to browser requests but are different security topics. CORS controls whether JavaScript can read a response; CSRF concerns unwanted state-changing requests made using a victim’s credentials.

## Simple Summary

> **CORS is a permission system that lets a server decide which other websites may read its responses through browser JavaScript.**

Remember the basic flow:

```
Website A sends request
        ↓
Server checks the Origin
        ↓
Server returns CORS headers
        ↓
Browser allows or blocks JavaScript from reading the response
```

The most important header to remember is:

```
Access-Control-Allow-Origin
```

It tells the browser which origin is allowed to access the response.

## Why can CORS become a security misconfiguration?

CORS itself is not a vulnerability. It becomes a problem when the server gives permission to **untrusted origins**, especially for sensitive or authenticated data.

A misconfigured CORS policy may allow an attacker-controlled website to make a browser request to a target application and read the response. If the victim is logged in, the response could contain private information. PortSwigger describes this risk as allowing a trusted or attacker-controlled domain to interact with the application in the logged-in user’s security context.

## Common CORS misconfigurations

### 1. Trusting every origin dynamically

A dangerous implementation is:

```
Access-Control-Allow-Origin: [value received in Origin]
```

If the server receives:

```
Origin: https://attacker.example
```

and responds:

```
Access-Control-Allow-Origin: https://attacker.example
Access-Control-Allow-Credentials: true
```

then the server has effectively trusted the attacker’s website. PortSwigger classifies trusting arbitrary origins as an overly permissive cross-domain policy.[[portswigger](https://portswigger.net/kb/issues/00200601_cross-origin-resource-sharing-arbitrary-origin-trusted)]

A secure server should compare the origin against a fixed allowlist:

```
https://app.example.com
https://admin.example.com
```

It should not automatically reflect every origin.

### 2. Allowing credentials for an untrusted origin

The following response is dangerous when used with sensitive data:

```
Access-Control-Allow-Origin: https://untrusted-site.example
Access-Control-Allow-Credentials: true
```

`Access-Control-Allow-Credentials: true` tells the browser that credentials such as cookies may be used in the cross-origin request.

This can allow an untrusted website to read authenticated responses if the server’s other controls are weak.[[zaproxy](https://www.zaproxy.org/docs/alerts/40040-3/)]

### 3. Trusting all subdomains

A policy such as this can create risk:

```
Allow all subdomains of example.com
```

This sounds safe, but if any subdomain is vulnerable, abandoned, user-controlled, or taken over, it may become a trusted origin. Trusting all subdomains increases the attack surface.[[portswigger](https://portswigger.net/kb/issues/00200603_cross-origin-resource-sharing-all-subdomains-trusted)]

For example:

```
https://app.example.com       trusted
https://old-test.example.com  vulnerable
https://attacker.example.com  controlled by attacker
```

A better approach is to allow only the exact origins that need access.

### 4. Using `null` carelessly

Some requests can have:

```
Origin: null
```

If the server responds:

```
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```

the policy may be unsafe, depending on the application and the data being exposed. `null` should not be trusted casually.

### 5. Allowing overly broad methods and headers

Example:

```
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE
Access-Control-Allow-Headers: *
```

This may give cross-origin applications more access than they actually need.

The server should allow only the required methods and headers:

```
Access-Control-Allow-Methods: GET
Access-Control-Allow-Headers: Content-Type
```

This is not a complete security boundary by itself, but reducing unnecessary permissions is safer.

## Important wildcard detail

This configuration is commonly described as dangerous:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

However, modern browsers do **not** allow the wildcard `*` to be used for credentialed CORS requests. The browser blocks that combination.[[developer.mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS/Errors/CORSNotSupportingCredentials)][[portswigger](https://portswigger.net/web-security/cors/access-control-allow-origin)]

This configuration:

```
Access-Control-Allow-Origin: *
```

may be acceptable for genuinely public, non-sensitive resources. It becomes inappropriate when the response contains private data or when credentials are required.

The more realistic dangerous pattern is usually:

```
Access-Control-Allow-Origin: [reflected attacker origin]
Access-Control-Allow-Credentials: true
```

## CORS is not the same as authentication

CORS does not decide whether a user is logged in. It only controls whether browser JavaScript from another origin can read a response.

The server must still implement:

- Authentication.
- Authorization.
- Session management.
- CSRF protection.
- Input validation.
- Rate limiting.

Also, CORS is mainly enforced by browsers. Tools such as Burp Suite, `curl`, or another server can send requests without being stopped by the browser’s CORS rules. Therefore, an API must never rely on CORS as its only access-control mechanism.

## Secure CORS example

If only one frontend needs access:

```
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Vary: Origin
```

The server should:

- Use an exact allowlist of trusted origins.
- Allow credentials only when necessary.
- Allow only required methods.
- Allow only required headers.
- Avoid trusting arbitrary origins.
- Avoid trusting every subdomain.
- Avoid exposing sensitive data through unnecessarily broad CORS rules.
- Use `Vary: Origin` when responses differ based on the requesting origin.

## Pentesting checklist

When assessing CORS in an authorized lab or bug-bounty scope:

1. Send a request containing an untrusted `Origin`.
2. Check whether the response returns that origin in `Access-Control-Allow-Origin`.
3. Check whether `Access-Control-Allow-Credentials: true` is present.
4. Determine whether the response contains sensitive information.
5. Test whether the policy trusts all subdomains or unusual origins.
6. Check whether preflight requests allow unnecessary methods or headers.
7. Confirm the behavior in a browser, because browser enforcement matters.
8. Do not report CORS merely because the header exists; demonstrate that sensitive data can actually be read by an unauthorized origin.
